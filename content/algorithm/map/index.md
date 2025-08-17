---
title: 使用map降低算法的时间复杂度
date: 2022-02-19T12:17:59+00:00
categories:
  - 算法与数据结构
tags:
  - map
---
<blockquote class="wp-block-quote">
  <p>
    在前两周在开发新接口时，遇到了今天要介绍的两种场景，我都通过map降低了整个算法的时间复杂度。我认为有必要记录一下思考过程，也顺便分享给各位读者。虽然简单，但确实很好用。
  </p>
</blockquote>

### <span class="ez-toc-section" id="1_%E7%BE%A4%E7%BB%84%E4%B8%AD%E7%9A%84%E5%A5%BD%E5%8F%8B"></span>1. 群组中的好友<span class="ez-toc-section-end"></span> {.wp-block-heading}

#### <span class="ez-toc-section" id="%E8%83%8C%E6%99%AF"></span>背景<span class="ez-toc-section-end"></span> {.wp-block-heading}

各位读者应该都知道在微信聊天框中输入@之后，会出现各群员的列表。新接口功能上差不多，只是在这基础上还要加上一个限制条件：“列表中的群成员必须是好友”。我可以从其他微服务已经有的“好友接口”获取某一个用户的好友列表(list)，那么我只需要找出和群成员列表的交集即可。

#### <span class="ez-toc-section" id="%E5%AE%9E%E7%8E%B0"></span>实现<span class="ez-toc-section-end"></span> {.wp-block-heading}

当时想到有两种方法可以实现上面说的效果（设群有N个群成员，M个好友）：

  1. 采用两层遍历查询方式，遍历群成员list + 二分查找好友list（查询的时间复杂度：O(N * logM)）
  2. 采用map，将群成员id作为map的key，role作为value（查询的时间复杂度：O(1)）

哪种方式更好？通过时间复杂度来看，答案就不言而喻了。部分代码如下：

<pre class="wp-block-code"><code lang="go" class="language-go line-numbers">// 获取群内成员（我的源码有用到role做别的业务逻辑处理）
func GetMemberInGroup(groupID int64) (map[int64]int8, error) 
   dataSet := make(map[int64]int8)
   strSql := "SELECT `player_id`, `role` FROM `friend_group` WHERE `group_id` = ? AND `type` = 1 AND `role` = 0"
   rows, err := db.MysqlCon.Raw(strSql, groupID).Rows()
   if err != nil {
      return nil, err
   }
   var (
      playerID int64
      role     int8
   )
   defer rows.Close()
   for rows.Next() {
      if err = rows.Scan(&playerID, &role); err == nil {
         dataSet[playerID] = role
      }
   }
   return dataSet, err
}</code></pre>

<pre class="wp-block-code"><code lang="go" class="language-go line-numbers">...
// 用其他微服务的接口获取`playerID`这个用户的好友列表（设长度为N1）
friends, err := model.ListFriend(playerID, int32(pb.&lt;em>FriendType_FT_FRIEND&lt;/em>))
if err != nil {
   ...
}
// 用上面的持久层方法获取群内成员
memberMap, err := model.GetMemberInGroup(groupID)
if err != nil {
   ...
}
// 结果（保存即是群员也是好友的用户id）
var result []int64
for _, friend := range friends {
   _, exists := memberMap[friend.FriendId]
   if exists {
      result = append(result, friend.FriendId)
   }
}
...</code></pre>

若各位读者要使用以上代码，最好在使用前能大致预测一下各组数据的大小，因为将数据多的list转为map，用数据小的list来做遍历会更好。

若其他微服务提供了获取好友map的接口，可以忽略上面的代码了。

### <span class="ez-toc-section" id="2_%E5%8B%8B%E7%AB%A0%E6%8E%92%E5%BA%8F"></span>2. 勋章排序<span class="ez-toc-section-end"></span> {.wp-block-heading}

#### <span class="ez-toc-section" id="%E8%83%8C%E6%99%AF-2"></span>背景<span class="ez-toc-section-end"></span> {.wp-block-heading}

需要将用户获得的勋章列表返回给客户端，且勋章列表需要管理员按照配置的权重排序。存在的问题：用户获得的勋章储存在数据库里（数据库里没有勋章的权重），勋章配置的结构体列表储存在Redis里（权重值存在Redis）。

#### <span class="ez-toc-section" id="%E5%AE%9E%E7%8E%B0-2"></span>实现<span class="ez-toc-section-end"></span> {.wp-block-heading}

部分代码如下：

<pre class="wp-block-code"><code lang="go" class="language-go line-numbers">// Redis的勋章配置结构体（权重值存在Redis）
// 这个是redis.Medal
type Medal struct {
   MedalId   string    `json:"column:Id"`
   MedalName string    `json:"Name"`
   CreateAt  time.Time `json:"CreateAt"`
   MedalType int32     `json:"Type"`
   MedalIcon string    `json:"Icon"`
   WeightInAchievePage int32  `json:"WeightInAchievePage"`
   ...
}

// 储存在数据库的用户获取的勋章（没有权重值）
// 这个是db.Medal
type Medal struct {
   MedalId   string    `gorm:"primary_key;column:Id"`
   CreateAt  time.Time `gorm:"column:CreateAt"`
   ...
}</code></pre>

<pre class="wp-block-code"><code lang="go" class="language-go line-numbers">...
// 从数据库查询用户获取的勋章
medalList, err := dao.MemberDB.GetMedalList()
var result []string
if err == nil {
   // 将用户获取的勋章list转map，减少查询的时间复杂度
   medalMap := make(map[string]*db.Medal, 0)
   for _, v := range medalList {
      medalMap[v.MedalId] = v
   }
   // 查询管理员后台页面的勋章配置列表
   // 这里medalConfList的类型是上面redis.Medal结构体数组
   // 值得一提的是这里redis.Medal结构体数组数据是在数据库按照权重排好序存进Redis的，代码在下面
   // 为了加快客户端获取配置的效率
   medalConfList := common.GetMedalList()
   for _, v := range medalConfList {
      if v.MedalType == int32(pb.&lt;em>MedalType_Glory&lt;/em>) {
         // 按照后台页面配置的勋章的权重，对已获取的勋章排序
         if _, ok := medalMap[v.MedalId]; ok {
            result = append(result, v.MedalName)
         }
      }
   }
}
...

func GetMedalList() []*redis.Medal {
   list := make([]*redis.Medal, 0)
   bin, err := db.RedisCon.Get(db.&lt;em>CacheMedalListKey&lt;/em>).Bytes()
   if err == nil && len(bin) &gt; 0 {
      if err = ujson.Unmarshal(bin, &list); err != nil {
         ...
      }
   }

   err = dao.MainDB().Order("WeightInAchievePage DESC").Find(&list).Error
   if err == nil || err == gorm.ErrRecordNotFound {
      if bin, err = ujson.Marshal(list); err == nil {
         db.RedisCon.Set(db.&lt;em>CacheMedalListKey&lt;/em>, bin, 0)
      }
   }
   return list
}
...</code></pre>

如果Redis存的勋章结构体里有储存勋章的权重，上面的代码就不适用了。

### <span class="ez-toc-section" id="%E7%BB%93%E8%AF%AD"></span>结语<span class="ez-toc-section-end"></span> {.wp-block-heading}

两个算法都是采用了类似的方法，用map这个数据结构，通过空间换时间来将查询时间复杂度降到了常数阶。如果读者有想到更好的算法，欢迎留言。
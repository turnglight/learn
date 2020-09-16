
+ 查看所有键,时间复杂度O(n),生产环境禁止使用
    + keys *
+ 键总数，时间复杂度O(1)
    + dbsize
+ 检查键是否存在
    + exists key
+ 删除键
    + del key
+ 键过期
    + expire key seconds(过期后会被自动删除)
        + ttl key(查看剩余过期时间)
            + >0:键剩余的过期时间
            + -1:键没有设置过期时间
            + -2:键不存在
+ 查看键的数据结构类型
    + type key


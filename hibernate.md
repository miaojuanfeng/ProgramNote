### Hibernate的悲观锁和乐观锁

悲观锁：通常依赖于数据库机制，在整修过程中将数据库锁定，其它任何用户都不能读取或修改，悲观锁一般是由数据库机制来做到的。sql中通常采用for update语句加锁。
在hibernate中使用悲观锁一般使用LockMode.UPGRADE，如果使用悲观锁，那么lazy(懒加载)无效。

乐观锁：不是锁，是一种冲突检测机制，乐观锁的并发性较好，因为我改的时候，别人可随便修改乐观锁的实现方式：常用的是版本的方式（每个数据表中有一个版本字段version,某一个用户更新数据库后，版本号+1,另一个用户修改后再+1,当用户更新发现数据库当前版本号与读取数据时版本号不一致，等于或小于数据库版本号则更新不了）
hibernate使用乐观锁需要在映射文件(*.hbm.xml)中配置才可生效。

### openSession与getCurrentSession的区别

openSession: 当调用SessionFactory的openSession方法时，它总是创建一个完全全新的session给你。你需要显示的刷新并且关闭session对象。因为session对象不是线程安全的，在多线程环境中你需要为每一个请求创建一个session对象（例如web应用的每一个请求）。

getCurrentSession: 当调用SessionFactory的getCurrentSession方法时，它会返回Hibernate上下文中的Session，并且有hibernate管理。它绑定到事物范围。当调用SessionFactory的getCurrentSession方法时，如果不存在他会创建一个新的Session对象，如果在当前的hibernate上下文中存在，则返回同样的Session对象。它会自动地flush和close，当事物结束的时候，所以无需多余处理。如果你在单线程环境下使用hibernate，你应该用getCurrentSession，它的性能比较好。

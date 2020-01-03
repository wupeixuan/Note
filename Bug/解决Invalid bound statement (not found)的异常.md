报错
```
org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): com.glodon.sync.dao.SyncDataLogDao.getLastUpdateReocrd
```

解决方法：

- 检查 SyncDataLogDao.xml 中的 namespace 是否与 SyncDataLogDao.java 一致
- 检查 SyncDataLogDao 接口和 SyncDataLogDao.xml 所在包名是否一致


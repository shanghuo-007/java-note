# synchronized业务场景

## 注册、签约验证场景

验证用户唯一性时使用`synchronized(accountNo.inern()){…}`

例：

```java
public void register(Account accout){
    // do validate
    synchronized(accout.getAccountNO().inern()){
        //do duplicates validate
        ...
        
        insert(account);
    }
}
```


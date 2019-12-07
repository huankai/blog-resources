---
title: Spring Data JPA 锁
date: {{ date }}
author: huangkai
tags:
    - Spring-Data-Jpa
---

乐观锁:
====
实体类:
----
```
package com.hk.mysql.examples.domain;

import com.hk.core.data.commons.typedef.JsonTypeDef;
import com.hk.core.data.jpa.domain.AbstractUUIDPersistable;
import com.vladmihalcea.hibernate.type.json.JsonStringType;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.experimental.Accessors;
import org.hibernate.annotations.Type;
import org.hibernate.annotations.TypeDef;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Table;
import javax.persistence.Version;
import java.io.Serializable;
import java.util.List;


/**
 * @author huangkai
 * @date 2019-02-26 17:22
 */
@Data
@Entity
@Accessors(chain = true)
@Table(name = "t_account")
@EqualsAndHashCode(callSuper = true)
@TypeDef(name = JsonTypeDef.json, typeClass = JsonStringType.class)
public class Account extends AbstractUUIDPersistable {

    @Column(name = "sheyuan_id")
    private String sheyuanId;

    @Column(name = "nick_name")
    private String nickName;

    @Type(type = "json")
    @Column(name = "content_one")
    private Content contentOne;

    /**
     * <pre>
     *
     * 使用 json 类型需要注意：
     * 1、需要添加依赖包 : https://github.com/vladmihalcea/hibernate-types
     * 2、要序列号的对象 {@link Content} 需要实现 {@link Serializable}
     * 3、加上 {@link Type} 注解，并在类上加上 {@link TypeDef} 注解
     * </pre>
     */
    @Column(name = "content")
    @Type(type = JsonTypeDef.json)
    private List<Content> content;

    /**
     * 使用版本注解实现 jpa 乐观锁
     */
    @Version
    @Column(name = "version")
    private Integer version;
}

```

数据库脚本:
------

```
CREATE TABLE `t_account` (
  `id` varchar(32) NOT NULL,
  `sheyuan_id` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `nick_name` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `content_one` json DEFAULT NULL,
  `content` json DEFAULT NULL,
  `version` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

INSERT INTO `hk_pms`.`t_account`(`id`, `sheyuan_id`, `nick_name`, `content_one`, `content`, `version`) VALUES ('402881196e889c58016e889c831b0000', NULL, 'dd3d66xxx', NULL, NULL, 0);

```

测试：
-----

```
@Test
public void update() {
    Account account = new Account();
    account.setId("402881196e889c58016e889c831b0000");
    account.setNickName("dd3d66xxx");
	/*
	设置版本号: 上面默认初始化的时候，数据库中的 version 值为 0，这里运行一次程序后，数据库的 version会自增,变成1
	执行的 Sql 语句为 :
		update t_account set content=?, content_one=?, nick_name=?, sheyuan_id=?, version=? where id=? and version=?
	则下次运行这个的时候，version 的值只能是 1才能更新成功，否则，会抛出 org.springframework.orm.ObjectOptimisticLockingFailureException		
	*/
    account.setVersion(0);
    accountService.insertOrUpdate(account);
}

```

以上方式只能适用于 调用 spring data jpa 的 save() 与 saveAll() 方法，如果你在 Repository 中自定义的方法，则姿势如下:

```
@Modifying
@Query("update t_account set nick_name=:name,version=:version+1 where id=:id and version=:version")
int updateAccountByVersion(@Param("id") int id,@Param("name") String name, @Param("version") int version);
```

service 方法调用 ：

```
@Transactional(rollbackFor = Exception.class)
    public boolean updateAccountByVersion(Account account){
        int rows =accountDao.updateAccountByVersion(account.getId(),account.getName(),account.getVersion());
        if(rows == 0){
            throw new ObjectOptimisticLockingFailureException("更新account失败",new Exception());
        }
        return true;
    }
```

悲观锁
===

参考链接 [https://www.jianshu.com/p/6f5e4108c71c](https://www.jianshu.com/p/6f5e4108c71c)
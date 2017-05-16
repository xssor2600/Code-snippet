## MySQL的树形查询<br>
在程序开发过程中，做权限管理或者其他包含树形层次结构的数据表的开发，通常都会有树形sql查询的需求。这里就想总结自己遇到与MySQL树形查询有关的内容,更希望自己可以不断的完善<br>

* 自上而下，查询节点所有子节点<br>
因为mysql不像oracle那般有现成的函数可以使用来查询子节点。那么只能自己写函数来查询了，通常情况下，结合树形的节点递归查询，配合mysql的自带的一些函数应该就差不多了。<br>

先来看看具有树形层次结构的权限表t_permission:<br>
```sql
CREATE TABLE `t_permission` (
  `n_id` bigint(15) NOT NULL AUTO_INCREMENT,
  `v_permission_name` varchar(30) NOT NULL,
  `v_url` varchar(100) DEFAULT NULL,
  `n_permission_STATE` tinyint(3) NOT NULL,
  `v_desc` varchar(150) DEFAULT NULL,
  `n_type` tinyint(3) DEFAULT NULL,
  `n_permission_level` tinyint(3) DEFAULT NULL,
  `n_parent_id` decimal(30,0) DEFAULT NULL,		//父节点id
  `n_menu_type` tinyint(2) NOT NULL,
  `n_create_staff_id` bigint(30) DEFAULT NULL,
  `v_create_time` varchar(50) DEFAULT NULL,
  `n_update_staff_id` bigint(30) DEFAULT NULL,
  `v_update_time` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`n_id`)
) ENGINE=InnoDB AUTO_INCREMENT=708 DEFAULT CHARSET=utf8

```
可以看到，在`t_permission`表中，每条记录中都有个**n_parent_id**字段，就是用来标识字节的父节点的。这样就形成了父子关系的树形节点集合。<br>
这样，当我们要查询某个在`t_permission`表中nid=xxx的节点的所有子节点(子节点下的子节点...递归)的时候，就可以使用如下函数:<br>
```sql
delimiter %%%%
CREATE FUNCTION
		getAllSubPermissions(parentId INT) RETURNS VARCHAR(100)
			BEGIN
					DECLEARE
						stemp VARCHAR(1000);
					DECLEARE
						stempChid VARCHAR(1000);
			SET stemp = '$';
			SET stempChid = CAST(parentId AS CHAR);
			WHILE stempChid IS NOT NULL DO
			SET stemp = CONCAT(stemp,',',stempChid);
			SELECT
				GROUP_CONCAT(n_id)
			INTO
				stempChid
			FROM
				t_permission
			WHERE
				FIND_IN_SET(n_parent_id,stempChid) > 0;
			END WHILE;
			RETURN stemp;
			END
%%%%		

```
那么，如何通过sql结合上述函数来实现查询呢，如下:<br>
```shell
##？即为要查询节点的n_id参数:即查询nid=??下所有的子节点的n_id集合
select n_id,t_permission.V_permission_NAME from t_permission where FIND_IN_SET(n_id,getAllSubPermissions(？));
```
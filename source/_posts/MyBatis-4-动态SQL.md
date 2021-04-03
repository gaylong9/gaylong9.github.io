---
title: MyBatis-4-动态SQL
mathjax: false
date: 2021-03-23 10:01:22
tags: 后端
---

&nbsp;

<!-- more -->

<!-- toc -->

&nbsp;

[toc]



# 动态SQL

## 1. 概述

包括以下元素，节省了许多拼接SQL的工作：

* if
* choose(when, otherwise)
* trim(where, set)
* foreach

&nbsp;

## 2. if元素

常与test属性联合使用。

eg：如角色名roleName为空就不用它作为查询条件

```xml
<select id="findRoles" parameterType="string" resultMap="roleResultMap">
    select role_no, role_name from t_role where 1=1
    <if test="roleName != null and roleName != ''">
        and role_name like concat('%', #{roleName}, '%')
    </if>
</select>
```

&nbsp;

## 3. choose, when, otherwise元素

eg：roleNo不空，就用roleNo查询；若roleNo空且roleName不空，就用roleName查询；若都空，就要求note不空

```xml
<select id="findRoles" parameterType="role" resultMap="roleResultMap">
    select role_no, role_name, note from t_role
    where 1=1
    <choose>
        <when test="roleNo != null and roleNo != ''">
            AND role_no = #{roleNo}
        </when>
        <when test="roleName != null and roleName != ''">
            AND role_name like concat('%', #{roleName}, '%')
        </when>
        <otherwise>
            AND note is not null
        </otherwise>
    </choose>
</select>
```

&nbsp;

## 4. trim, where, set元素

1. where：上例的`1=1`是为了避免SQL错误，可用where元素替代，where元素知道只有在一个以上的if条件有值的情况下才去插入“WHERE”子句。而且，若最后的内容是“AND”或“OR”开头的，where 元素也知道如何将他们去除。

	```xml
	<select ...>
	    select role_no, role_name, note from t_role 
	    <where>
	        <if test="roleName != null and roleName != ''">
		        and role_name like concat('%', #{roleName}, '%')
		    </if>
	        <if test="note != null and note !=''">
	        	and note like concat('%', #{note}, '%')
	        </if>
	    </where>
	</select>
	```

2. trim：去掉一些特殊的SQL语法，如and or；有prefix和suffix表示前后缀；下例实现了与where元素相同的功能，若判断成功就前缀where并删去SQL前缀的and

	```xml
	<select ...>
	    select role_no, role_name from t_role
	    <trim prefix="where" preifxOverrides="and">
	        <if test="roleName != null and roleName != ''">
	            and role_name like concat('%', #{roleName}, '%')
	        </if>
	    </trim>
	</select>
	```

3. set：会动态前置SET关键字，同时也会消除无关的逗号

	```xml
	<update id="updateRole" parameterType="role">
	    update t_role
	    <set>
	        <if test="roleName != null and roleName != ''">
	            role_name = #{roleName},
	        </if>
	        <if test="note != null and note != ''">
	            note = #{note}
	        </if>
	    </set>
	    where role_no = #{roleNo}
	</update>
	```

	转换为trim:

	```xml
	<trim prefix="SET" suffixOverrides=","> ... </trim>
	```

	&nbsp;

## 5. foreach元素

遍历集合，用于in关键字

```xml
<select id="" resultType="user">
    select * from t_role where role_no in
    <foreach item="roleNo" index="index" collection="roleNoList" open="(" separator="," close=")">
        #{roleNo}
    </foreach>
</select>
```

collection；配置传入的参数

item：循环中的当前元素

index：当前元素的下标

open和close：以什么符号包装集合元素

&nbsp;

## 6. test属性判断字符串

前例中test用于判断非空，还可用于判断字符串。

```xml
<select id="" parameterType="string" resultMap="roleResultMap">
    select role_no, role_name from t_role
    <if test="type == 'Y'.toString()">
        where 1=1
    </if>
</select>
```

此时如果传入type==‘Y’，则会判断成功，加入where 1=1子句。

test可判断数值型参数。

枚举的话，取决于使用何种typeHandler。

&nbsp;

## 7. bind元素

通过OGNL表达式定义一个上下文变量，方便使用。如模糊查询时，MySQL使用concat，Oracle用其他方法。bind元素提供统一方法，不必为不同数据库配置不同SQL。

eg：按角色名进行模糊查询

```xml
<select id parameterType="string" resultType="RoleBean">
    <bind name="pattern" value="'%' + _parameter + '%'"/>
    select id, role_name as roleName from t_role where role_name like #{pattern}
</select>
```

_parameter：代表传入的参数，和通配符连接后赋给pattern。






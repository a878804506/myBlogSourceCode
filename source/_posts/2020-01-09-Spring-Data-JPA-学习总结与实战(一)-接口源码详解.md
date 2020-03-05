---
title: Spring Data JPA 学习总结与实战(一)-接口源码详解
top: true
cover: true
date: 2020-01-09 14:09:32
categories: Spring Data JPA
tags:
  - spring全家桶
  - 框架
  - 总结
---

### 写在前面
1. 在17年的时候，我参与了一个项目用的就是spring data jpa，那时候对hebernate不是很了解（因为我一直都是用的mybatis），稀里糊涂的做着需求，好在spring data jpa入门还是很容易的，特别的简单查询，就这样在百度中参与进那个项目的开发中；现在刚好有点时间可以学一学，就总结一下我的学习成果。
2. 本系列文章是我在学习了《Spring data JPA从入门到精通》之后的总结；最后一篇系列文章将会有我写的例子作为实战，来检验一下自己的学习成果。
3. [**<font color=purple>《Spring data JPA从入门到精通》电子书下载</font>**](http://staticfile.erdongchen.top/download/Spring-Data-JPA从入门到精通.pdf?n=Spring_data_JPA从入门到精通.pdf "点击下载")

### 一、本篇教程侧重点导读
1. 顶级接口Repostitory介绍及层级关系；
2. CrudRepository接口方法详解；
3. PagingAndSortingRepository接口方法详解；
4. JpaRepository接口方法详解；
5. Repository的实现类SimpleJpaRepository介绍；
6. 自定义查询方法命名规则；
7. 查询方法关键字；

### 二、本篇教程用的软件、技术和说明
1. spring boot 版本：2.1.1.RELEASE；
2. Spring Data JPA 版本：2.1.3.RELEASE；

### 三、顶级接口Repostitory介绍及层级关系
1. 首先要知道jpa是一种规范，hebernate是jpa的一种实现，Spring Data JPA底层用的还是hebernate，Spring Data JPA 是Spring Data的一个子项目。它依赖了Spring Data Common包,而接口Repostitory也是位于Spring Data Common的lib里面的，是Spring Data里面做数据库操作的最底层的抽象接口、 最顶级的父类， 源码里面其实什么方法都没有， 仅仅起到一个标识作用。
Repostitory的源码如下：
````java
 package org.springframework.data.repository;

 import org.springframework.stereotype.Indexed;

 @Indexed
 public interface Repository<T, ID> {

 }
````
2. 我们可以利用Idea来查看Repostitory的层级关系，具体如下：
 ①. 打开Repository.class，该接口位于org.springframework.data.repository包下；
 ②. 快捷键Ctrl+H，如图所示：
 <img style="width:85%;height:85%" src="http://staticfile.erdongchen.top/blog/blogPicture/20200109/3.1.png"  align=left/>

### 四、CrudRepository接口方法详解
1. CrudRepository的源码如下：
````java
 package org.springframework.data.repository;

 import java.util.Optional;

 @NoRepositoryBean
 public interface CrudRepository<T, ID> extends Repository<T, ID> {
	<S extends T> S save(S entity);
	<S extends T> Iterable<S> saveAll(Iterable<S> entities);//批量保存
	Optional<T> findById(ID id);
	boolean existsById(ID id);
	Iterable<T> findAll();
	Iterable<T> findAllById(Iterable<ID> ids);
	long count();//计算对象总个数
	void deleteById(ID id);
	void delete(T entity);
	void deleteAll(Iterable<? extends T> entities);
	void deleteAll();
}
````
2. 该接口主要是完成一些增删改查的操作，例如查看save(S entity)方法的具体实现，快捷键Ctrl+Alt+鼠标点击方法名save；可以看到底层实现（实现类是SimpleJpaRepository）：
````java
 @Transactional
 public <S extends T> S save(S entity) {
     if (entityInformation.isNew(entity)) {
         em.persist(entity);
         return entity;
     } else {
         return em.merge(entity);
     }
 }
````
3. 我们发现它是先检查传进去的实体是不是存在， 然后判断是新增还是更新； 是不是存在两种根据机制， 一种是根据主键来判断， 另一种是根据Version来判断。如果我们去看JPA控制台打印出来的SQL， 最少会有两条， 一条是查询， 一条是insert或者update。类似的删除方法也是一样的，程序会先判断存不存在，再去删除，所以这里特别强调一下delete和save方法， 因为在实际工作中有的人会画蛇添足， 自己先去查询再做判断处理， 其实Spring JPA底层都已经考虑到了。

### 五、PagingAndSortingRepository接口方法详解
1. 话不多说，先上源码：
````java
 package org.springframework.data.repository;

 import org.springframework.data.domain.Page;
 import org.springframework.data.domain.Pageable;
 import org.springframework.data.domain.Sort;

 @NoRepositoryBean
 public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {
	Iterable<T> findAll(Sort sort);//排序
	Page<T> findAll(Pageable pageable);//分页并排序
 }
````
2. findAll(Pageable pageable)方法根据分页和排序进行查询， 并用Page对象封装。 Pageable对象包含分页和Sort对象。PagingAndSortingRepository和CrudRepository都是Spring Data   Common的标准接口， 如果我们采用JPA， 那它对应的实现类就是Spring Data JPA的model里面的SimpleJpaRepository。

### 六、JpaRepository接口方法详解
1. JpaRepository源码如下：
````java
 package org.springframework.data.jpa.repository;

 import java.util.List;

 import javax.persistence.EntityManager;

 import org.springframework.data.domain.Example;
 import org.springframework.data.domain.Sort;
 import org.springframework.data.repository.NoRepositoryBean;
 import org.springframework.data.repository.PagingAndSortingRepository;
 import org.springframework.data.repository.query.QueryByExampleExecutor;

 @NoRepositoryBean
 public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
     List<T> findAll();
     List<T> findAll(Sort sort);
     List<T> findAllById(Iterable<ID> ids);
     <S extends T> List<S> saveAll(Iterable<S> entities);
     void flush();//强制缓存与数据库同步
     <S extends T> S saveAndFlush(S entity);
     void deleteInBatch(Iterable<T> entities);//保存并强制同步数据库
     void deleteAllInBatch();
     T getOne(ID id);
     @Override
     <S extends T> List<S> findAll(Example<S> example);//根据实例查询
     @Override
     <S extends T> List<S> findAll(Example<S> example, Sort sort);//根据实例查询并排序
 }
````

2. 通过源码和CrudRepository相比较， 它支持Query By Example，批量删除， 提高删除效率， 手动刷新数据库的更改方法， 并将默认实现的查询结果变成了List。

### 七、Repository的实现类SimpleJpaRepository介绍
1. 源码太多了，略过；首先看一下SimpleJpaRepository的类构图
<img style="width:85%;height:85%" src="http://staticfile.erdongchen.top/blog/blogPicture/20200109/7.1.png"  align=left/>
2. ①. SimpleJpaRepository实现了JpaRepositoryImplementation接口，它是CrudRepository的默认实现；它的构造器都要求传入EntityManager
   ②. 它的类上注解了@Transactional(readOnly = true)；而对deleteById、delete、deleteAll、deleteInBatch、deleteAllInBatch、save、saveAndFlush、saveAll、flush都添加了@Transactional注解
   ③. 从各个方法的实现可以看到SimpleJpaRepository是使用EntityManager来完成具体的方法功能，对于查询功能很多都借助了applySpecificationToCriteria方法，将spring data的Specification转换为javax.persistence的CriteriaQuery

### 八、自定义查询方法命名规则；
1. 主要接口看完了，除了上述接口中自带的方法外，日常开发中还需用到其他的查询，如根据名称查询，模糊查询，日期区间查询等等，具体如下：
2. 例如我有个业务接口叫StudentRepository继承自JpaRepository接口；那么我就可以在这个业务接口里面定义一个方法`List<Student> findByNameLike(String name);`，按照一定的规则去命名接口，Spring Data JPA 就会自动根据方法名称去生成sql，返回数据，怎么样？很吊吧？
3. 查看org.springframework.data.repository.query.parser.PartTree源码可以看到，Spring Data JPA 在解析业务接口的时候不光是find能解析成查询语句，还有read|get|query|stream都可以作为前缀
````java
    private static final String QUERY_PATTERN = "find|read|get|query|stream";
    private static final String COUNT_PATTERN = "count";
    private static final String EXISTS_PATTERN = "exists";
    private static final String DELETE_PATTERN = "delete|remove";
````

### 九、查询方法关键字；
 最后附上一份接口方法命名规则：
 
 |序号|关键字|SQL符号|方法命名样例|对应JPQL语句片段|
 |:-:|:---:|:---:|:---------:|:---------:|
 |1|And|	and|findByLastnameAndFirstname|… where x.lastname = ?1 and x.firstname = ?2|
 |2|Or|or|findByLastnameOrFirstname|… where x.lastname = ?1 or x.firstname = ?2|
 |3|Is,Equals|=|findByFirstname,findByFirstnameIs|… where x.firstname = ?1|
 |4|Between|between xxx and xxx|findByStartDateBetween|… where x.startDate between ?1 and ?2|
 |5|LessThan|<|findByAgeLessThan|… where x.age < ?1|
 |6|LessThanEqual|<=|findByAgeLessThanEqual|… where x.age <= ?1|
 |7|GreaterThan|>|findByAgeGreaterThan|… where x.age > ?1|
 |8|GreaterThanEqual|>=|findByAgeGreaterThanEqual|… where x.age >= ?1|
 |9|After|>|findByStartDateAfter|… where x.startDate > ?1|
 |10|Before|<|findByStartDateBefore|… where x.startDate < ?1|
 |11|IsNull|is null|findByAgeIsNull|… where x.age is null|
 |12|IsNotNull,NotNull|is not null|findByAge(Is)NotNull|… where x.age not null|
 |13|Like|like|findByFirstnameLike|… where x.firstname like ?1|
 |14|NotLike|not like|findByFirstnameNotLike|… where x.firstname not like ?1|
 |15|StartingWith|like 'xxx%'|findByFirstnameStartingWith|… where x.firstname like ?1(parameter bound with appended %)|
 |16|EndingWith|like 'xxx%'|findByFirstnameEndingWith|… where x.firstname like ?1(parameter bound with prepended %)|
 |17|Containing|like '%xxx%'|findByFirstnameContaining|… where x.firstname like ?1(parameter bound wrapped in %)|
 |18|OrderBy|order by|findByAgeOrderByLastnameDesc|… where x.age = ?1 order by x.lastname desc|
 |19|Not|<>|findByLastnameNot|… where x.lastname <> ?1|
 |20|In|in()|findByAgeIn(Collection<Age> ages)|… where x.age in ?1|
 |21|NotIn|not in()|findByAgeNotIn(Collection<Age> ages)|… where x.age not in ?1|
 |22|TRUE|=true|findByActiveTrue()|… where x.active = true|
 |23|FALSE|=false|findByActiveFalse()|… where x.active = false|
 |24|IgnoreCase|upper(xxx)=upper(yyyy)|findByFirstnameIgnoreCase|… where UPPER(x.firstame) = UPPER(?1)|

















# Jetpack-Room



room是在编译阶段生成查询代码的，且会自动在查询的过程中更新所引用的LiveData值。使用订阅的方式（invalidateTracker）获取到改变，再通过CAS的方式去完成数据库查询并postValue。

## 使用Room实体定义数据

1.定义类，并给类添加注解（Entity）

​		要保留某个字段，Room 必须拥有该字段的访问权限。您可以将某个字段设为公开字段，也可以为其提供 getter 和 setter。如果使用 getter 和 setter 方法，请注意，这些方法需遵循 Room 中的 JavaBeans 规范。？？？ps：暂未尝试，还需后面实现

2.设置主键（primaryKey/primaryKeys）：Sqlite中表名称不区分大小写

3.设置每一个列的名称：ColumnInfo。也可以不设置，会有默认值

4.默认情况下，Room 会为实体中定义的每个字段创建一个列。如果某个实体中有您不想保留的字段，则可以使用 Ignore为这些字段添加注释。如果实体继承了父实体的字段，则使用 `@Entity` 属性的 ignoredColumns属性通常会更容易

5.通常设置完相应的类则是设置相对应的dao，最后设置相对应的roomdatabase，会自动根据上述的实体类设置代码。

6.如果表支持以多种语言显示的内容，请使用 `languageId` 选项指定用于存储每一行语言信息的列

## 全文搜索

​	全文搜索是从文本或数据库中，不限定数据字段自由地搜索出信息的技术。它的工作原理是扫描文章中的每一个词，对每一个词建立一个索引，指明该词在文章中出现的次数和位置，当进行查询时，数据库就根据事先建立的索引进行查找，从而大幅度加快查找速度。SQLite 支持全文搜索，目前常用的版本是 FTS3 和 FTS4，最新的 FTS5 版本需要 SQLite3.9.0。

### 在 ROOM 中使用 FTS（full text search）

使用 @Fts3 或 @Fts4 注释带有 @Entity 注释的类，注意：

1. 使用 @Fts3 或 @Fts4 注释后，Entity 就不能有外键
2. 主键必须为 `rowid` 或使用 @ColumnInfo(name = "rowid")
3. 查询时返回的实体不会包含 rowid。

### FTS3 和 FTS4 的区别

1. FTS4 包含了查询优化，可以显著提升高频词的检索性能
2. FTS4 支持 hooks 来实现压缩存储以减小磁盘开销。

## 将特定列编入索引

如果您的应用必须支持不允许使用由 FTS3 或 FTS4 表支持的实体的 SDK 版本，您仍可以将数据库中的某些列编入索引，以加快查询速度。如需为实体添加索引，请在 [`@Entity`](https://developer.android.com/reference/androidx/room/Entity) 注释中添加 [`indices`](https://developer.android.com/reference/androidx/room/Entity#indices()) 属性，列出要在索引或复合索引中包含的列的名称。

```java
@Entity(indices = {@Index("name"),
            @Index(value = {"last_name", "address"})})
    public class User {
        @PrimaryKey
        public int id;

        public String firstName;
        public String address;

        @ColumnInfo(name = "last_name")
        public String lastName;

        @Ignore
        Bitmap picture;
    }
```

有时，数据库中的某些字段或字段组必须是唯一的。您可以通过将 [`@Index`](https://developer.android.com/reference/androidx/room) 注释的 [`unique`](https://developer.android.com/reference/androidx/room#unique()) 属性设为 `true`，强制实施此唯一性属性。

```java
@Entity(indices = {@Index(value = {"first_name", "last_name"},
            unique = true)})
    public class User {
        @PrimaryKey
        public int id;

        @ColumnInfo(name = "first_name")
        public String firstName;

        @ColumnInfo(name = "last_name")
        public String lastName;

        @Ignore
        Bitmap picture;
    }
```

## 添加基于 AutoValue 的对象

在 Room 2.1.0 及更高版本中，可以将基于 Java 的[不可变值类](https://github.com/google/auto/blob/master/value/userguide/index.md)（使用 `@AutoValue` 进行注释）用作应用数据库中的实体。此支持在实体的两个实例被视为相等（如果这两个实例的列包含相同的值）时尤为有用。

将带有 `@AutoValue` 注释的类用作实体时，可以使用 `@PrimaryKey`、`@ColumnInfo`、`@Embedded` 和 `@Relation` 为该类的抽象方法添加注释。但是，必须在每次使用这些注释时添加 `@CopyAnnotations` 注释，以便 Room 可以正确解释这些方法的自动生成实现。

## 使用RoomDAO访问数据

这些 [`Dao`](https://developer.android.com/reference/androidx/room/Dao) 对象构成了 Room 的主要组件，因为每个 DAO 都包含一些方法，这些方法提供对应用数据库的抽象访问权限。

通过使用 DAO 类（而不是查询构建器或直接查询）访问数据库，您可以拆分数据库架构的不同组件。DAO **既可以是接口，也可以是抽象类**。如果是**抽象类**，则该 DAO 可以选择有一个以 [`RoomDatabase`](https://developer.android.com/reference/androidx/room/RoomDatabase) 为唯一参数的构造函数。**Room 会在编译时创建每个 DAO 实现。**（room自动完成）

### 定义方法以方便实用

#### insert

创建 DAO 方法并使用 [`@Insert`](https://developer.android.com/reference/androidx/room/Insert) 对其进行注释时，Room 会生成一个实现，该实现在单个事务中将所有参数插入数据库中。

```java
 @Dao
    public interface MyDao {
        @Insert(onConflict = OnConflictStrategy.REPLACE)
        public void insertUsers(User... users);

        @Insert
        public void insertBothUsers(User user1, User user2);

        @Insert
        public void insertUsersAndFriends(User user, List<User> friends);
    }
    如果 @Insert 方法只接收 1 个参数，则它可以返回 long，这是插入项的新 rowId。如果参数是数组或集合，则应返回 long[] 或 List<Long>。
```

#### update

便捷方法会修改数据库中以参数形式给出的一组实体。它使用与每个实体的主键匹配的查询。

```java
@Dao
    public interface MyDao {
        @Update
        public void updateUsers(User... users);
    }
    虽然通常没有必要，但是可以让此方法返回一个 int 值，以指示数据库中更新的行数。
```

#### 删除

便捷方法会从数据库中删除一组以参数形式给出的实体。它使用主键查找要删除的实体。代码同上。虽然通常没有必要，但是您可以让此方法返回一个 `int` 值，以指示从数据库中删除的行数。

#### 查询信息

Query是 DAO 类中使用的主要注释。它允许您对数据库执行读/写操作。每个 Query方法都会在**编译时**进行验证，因此如果查询出现问题，则会发生**编译错误**，而不是运行时失败。

Room 还会验证查询的返回值，以确保当返回的对象中的字段名称与查询响应中的对应列名称不匹配时，Room 可以通过以下两种方式之一提醒您：

- 如果只有部分字段名称匹配，则会发出警告。

- 如果没有任何字段名称匹配，则会发出错误。

##### 简单查询

  ```java
   @Query("SELECT * FROM user")
  ```

##### 将参数传递给查询

  ```java
  @Dao
      public interface MyDao {
          @Query("SELECT * FROM user WHERE age > :minAge")
          public User[] loadAllUsersOlderThan(int minAge);
      }
  编译时处理此查询时，Room 会将 :minAge 绑定参数与 minAge 方法参数进行匹配。Room 通过参数名称进行匹配。如果有不匹配的情况，则应用编译时会出现错误。
    
  ```

##### 返回列的子集

借助 Room，可以从查询中返回任何基于 Java 的对象，前提是结果列集合会映射到返回的对象。

##### 直接光标访问（强烈不建议）

如果应用的逻辑要求直接访问返回行，可以从查询中返回 `Cursor` 对象

```java
@Dao
    public interface MyDao {
        @Query("SELECT * FROM user WHERE age > :minAge LIMIT 5")
        public Cursor loadRawUsersOlderThan(int minAge);
    }
    强烈建议不要使用 Cursor API，因为它无法保证行是否存在或者行包含哪些值。只有当已具有需要光标且无法轻松重构的代码时，才使用此功能。
```

##### 使用 LiveData 进行可观察查询

执行查询时，您通常会希望应用的界面在数据发生变化时自动更新。为此，请在查询方法说明中使用 [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData) 类型的返回值。当数据库更新时，Room 会生成更新 [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData) 所必需的所有代码。

### 测试和调试数据库

1.在Android设备上测试

推荐的方法是编写在 Android 设备上运行的 JUnit 测试。由于执行这些测试不需要创建 Activity，因此它们的执行速度应该比界面测试速度快。在设置测试时，应创**建内存版本的数据库**以使测试更加封闭

```java
@RunWith(AndroidJUnit4.class)
public class SimpleEntityReadWriteTest {
    private UserDao userDao;
    private TestDatabase db;

    @Before
    public void createDb() {
        Context context = ApplicationProvider.getApplicationContext();
        db = Room.inMemoryDatabaseBuilder(context, TestDatabase.class).build();
        userDao = db.getUserDao();
    }

    @After
    public void closeDb() throws IOException {
        db.close();
    }

    @Test
    public void writeUserAndReadInList() throws Exception {
        User user = TestUtil.createUser(3);
        user.setName("george");
        userDao.insert(user);
        List<User> byName = userDao.findUsersByName("george");
        assertThat(byName.get(0), equalTo(user));
    }
}
```

2.在主机开发计算机上测试（不测试）

使用方法：在测试方法题内部，“右键”，在弹出菜单中选择“Run 方法名”即可（单元测试也可以使用断点调试和性能调试）。

3.调试数据库

​	Android SDK 包含一个 `sqlite3` 数据库工具，可用于检查应用的数据库。它包含用于输出表格内容的 `.dump` 以及用于输出现有表格的 `SQL CREATE` 语句的 `.schema` 等命令。

也可以从命令行执行 SQLite 命令，如以下代码段所示：（不是非常了解）

```bsh
adb -s emulator-5554 shell
sqlite3 /data/data/your-app-package/databases/rssitems.db
```

# PS

关于kotlin和RxJAVA相关的没有看
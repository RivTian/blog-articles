> Rust 版本：1.41.0 (1.24 以上)
> diesel 版本 1.4.3
> 使用MySQL进行模块

参考

* [repo](https://github.com/diesel-rs/diesel)
* [官网](https://diesel.rs/)
* [API Doc](http://docs.diesel.rs/diesel/index.html)
* [Custom types in Diesel](https://kitsu.me/posts/2020_05_24_custom_types_in_diesel)

## 一、Getting Started

### 1、简介

原则

* 防止运行时错误
* 性能
* 生产性和可扩展性
    * 与Active Record和其他ORM不同，Diesel旨在进行抽象。Diesel使您可以编写可重用的代码，并根据问题域而不是SQL进行思考。

支持的数据库

* [MySQL](https://docs.diesel.rs/diesel/pg/index.html)
* [PostgreSQL](https://docs.diesel.rs/diesel/mysql/index.html)
* [SQLite](https://docs.diesel.rs/diesel/sqlite/index.html)

### 2、创建测试项目和依赖

`cargo new --lib diesel-learn`

依赖，相关 [features](https://github.com/diesel-rs/diesel/blob/master/diesel/Cargo.toml)

```toml
[dependencies]
# diesel = { version = "1.4", features = ["postgres"] }
diesel = { version = "1.4", features = ["mysql"] }
# diesel = { version = "1.4", features = ["sqlite"] }
dotenv = "0.15.0"
```

### 3、Hello World

#### （1）创建MySQL连接工厂函数配置数据库连接

`src/lib.rs`

```rust
#[macro_use]
extern crate diesel;
extern crate dotenv;

use diesel::prelude::*;
use diesel::mysql::MysqlConnection;
use dotenv::dotenv;
use std::env;

// 建立连接
pub fn establish_connection() -> MysqlConnection {
    dotenv().ok();

    // 从数据库中拿到环境变量
    let database_url = env::var("DATABASE_URL")
        .expect("DATABASE_URL must be set");
    // 建连MySQL连接
    MysqlConnection::establish(&database_url)
        .expect(&format!("Error connecting to {}", database_url))
}

```

配置连接 `.env`

```env
DATABASE_URL=mysql://root:123456@localhost/diesel_learn
```

#### （2）创建两个模块

* `models` 模型
* `schema` 对数据表结构的描述，代码一般通过命令行工具 `diesel_cli` 创建，用于构建SQL

`src/lib.rs`

```rust
// ...

pub mod schema;
pub mod models;
```

创建两个文件

* `src/models.rs`
* `src/schema.rs`

编写Model，`src/models.rs`

```rust
use super::schema::posts;

// 用于查询
#[derive(Queryable, Debug)]
pub struct Post {
    pub id: i64,
    pub title: String,
    pub body: String,
    pub published: bool,
}

// 用于创建
#[derive(Insertable)]
#[table_name="posts"]
pub struct NewPost<'a> {
    pub title: &'a str,
    pub body: &'a str,
}

// 伪装成一张表，为了获取最新插入的自动自增的ID （MySQL特有）
table! {
    sequences(id) {
        id -> BigInt,
    }
}

// 用于获取id
#[derive(QueryableByName)]
#[table_name="sequences"]
pub struct Sequence {
    pub id: i64,
}
```

#### （3）使用diesel_cli工具创建`schema`

参见下文 [安装diesel_cli](二、diesel_cli（migration）工具)

初始化 `diesel_cli` 工具

```bash
diesel setup
```

配置 `diesel_cli` 工具配置文件 `diesel.toml`，生成代码附带 docs

```toml
# For documentation on how to configure this file,
# see diesel.rs/guides/configuring-diesel-cli

[print_schema]
file = "src/schema.rs"  # schema文件路径
with_docs = true  # 是否创建Docs注释
```

创建 数据库变更

```bash
diesel migration generate create_posts
```

编写 `migrations/2020-03-03-133828_create_posts/up.sql` （变更）

```sql
-- Your SQL goes here
CREATE TABLE posts (
  id bigint PRIMARY KEY auto_increment comment 'ID',
  title VARCHAR(256) NOT NULL comment '标题',
  body TEXT NOT NULL comment '内容',
  published BOOLEAN NOT NULL DEFAULT 0 comment '是否发布'
)
```

编写 `migrations/2020-03-03-133828_create_posts/down.sql` （回滚）

```sql
-- This file should undo anything in `up.sql`
DROP TABLE posts
```

执行数据库变更同时更新`src/schema.rs`

```bash
diesel migration run
```

此时 `src/schema.rs` 文件如下

```rust
table! {
    posts (id) {
        id -> Bigint,
        title -> Varchar,
        body -> Text,
        published -> Bool,
    }
}
```

同时数据库中该表已经创建完成

由此可以看出 diesel 原理

* 使用 `diesel_cli` 工具将数据库 表结构 转换为 `diesel` 的宏语法，并生成代码写到 `schema.rs` 文件中
* 调用 `schema.rs` 文件构建查询，最后通过 `.load::<Model>` 等一系列函数执行SQL序列换为Model，diesel 负责生成SQL，并反序列化为类型实例

#### （4）查询

`src/bin/show_posts.rs`

```rust
extern crate diesel_learn;
extern crate diesel;

use self::diesel_learn::*;
use self::models::*;
use self::diesel::prelude::*;

// cargo run --bin show_posts
fn main() {
    // 导入代码
    use diesel_learn::schema::posts::dsl::*;

    // 创建连接
    let connection = establish_connection();
    // 查询
    let results = posts.filter(published.eq(true))
        .limit(5)
        .load::<Post>(&connection)
        .expect("Error loading posts");

    // 打印
    println!("Displaying {} posts", results.len());
    for post in results {
        println!("{:?}", post);
    }
}
```

#### （5）插入

`src/lib.rs`

```rust
// ...

use self::models::{Sequence, NewPost};

pub fn create_post<'a>(conn: &MysqlConnection, title: &'a str, body: &'a str) -> i64 {
    use schema::posts;

    // 构建待插入对象
    let new_post = NewPost {
        title: title,
        body: body,
    };

    // 插入到数据库
    diesel::insert_into(posts::table)
        .values(&new_post)
        // .get_result(conn) // MySQL 不支持
        .execute(conn)
        .expect("Error saving new post");

    // 获取到id
    let generated_id = diesel::sql_query("select LAST_INSERT_ID() as id")
        .load::<Sequence>(conn).expect("get_id_error").first().unwrap().id;
    generated_id
}
```

`src/bin/write_post.rs`

```rust
extern crate diesel_learn;
extern crate diesel;

use self::diesel_learn::*;
use std::io::{stdin, Read};

//cargo run --bin write_post
fn main() {
    // 创建连接
    let connection = establish_connection();

    // 获取用户输入
    println!("What would you like your title to be?");
    let mut title = String::new();
    stdin().read_line(&mut title).unwrap();
    let title = &title[..(title.len() - 1)]; // Drop the newline character
    println!("\nOk! Let's write {} (Press {} when finished)\n", title, EOF);
    let mut body = String::new();
    stdin().read_to_string(&mut body).unwrap();

    // 执行插入
    let id = create_post(&connection, title, &body);
    println!("\nSaved draft {} with id {}", title, id);
}

#[cfg(not(windows))]
const EOF: &'static str = "CTRL+D";

#[cfg(windows)]
const EOF: &'static str = "CTRL+Z";
```

#### （6）更新

`src/bin/publish_post.rs`

```rust
extern crate diesel_learn;
extern crate diesel;

use self::diesel::prelude::*;
use self::diesel_learn::*;
use self::models::Post;
use std::env::args;

// cargo run --bin publish_post 1
fn main() {
    use diesel_learn::schema::posts::dsl::{posts, published, id};

    let id_value = args().nth(1).expect("publish_post requires a post id")
        .parse::<i64>().expect("Invalid ID");
    let connection = establish_connection();

    diesel::update(posts.filter(id.eq(id)))
        .set(published.eq(true))
        // .get_result::<Post>(&connection)
        .execute(&connection)
        .unwrap();

    let post = posts
        .find(id_value)
        .first::<Post>(&connection)
        .expect(&format!("Unable to find post {}", id_value));
    println!("Published post {:?}", post);
}
```

#### （7）删除

```rust
extern crate diesel_learn;
extern crate diesel;

use self::diesel::prelude::*;
use self::diesel_learn::*;
use std::env::args;

// cargo run --bin delete_post test
fn main() {
    use diesel_learn::schema::posts::dsl::*;

    let target = args().nth(1).expect("Expected a target to match against");
    let pattern = format!("%{}%", target);

    let connection = establish_connection();
    let num_deleted = diesel::delete(posts.filter(title.like(pattern)))
        .execute(&connection)
        .expect("Error deleting posts");

    println!("Deleted {} posts", num_deleted);
}
```

## 二、diesel_cli（migration）工具

参考：https://github.com/diesel-rs/diesel/tree/master/diesel_cli

Diesel提供了单独的CLI工具来帮助您管理项目。由于它是一个独立的二进制文件，并且不会直接影响您项目的代码，因此我们不会将其添加到Cargo.toml中。相反，我们只是将其安装在系统上。

`diesel_cli` 主要功能如下

* 管理 `migration` 执行/回滚
* 创建/更新 `schema.rs` 文件，生成代码

### 1、安装

```bash
cargo install diesel_cli
```

### 2、数据库配置

使用 `DATABASE_URL` 环境变量或者 `--database-url=xxx` 命令行参数

推荐使用 `.env` 环境变量文件

```bash
# echo DATABASE_URL=[mysql|postgres|sqlite]://username:password@localhost/db_name > .env
echo DATABASE_URL=mysql://root:123456@localhost/diesel_learn > .env
```

### 3、使用

* 初始化配置 `diesel setup`
    * 创建 `migrations` 目录
    * 创建 `diesel.toml` 配置文件
    * 创建数据库（不存在的话）
    * 在数据库中创建 `__diesel_schema_migrations` 数据表
        * 该表是 `migration` 跟踪表记录了当前schema的版本
    * 执行所有未执行的 `migration`
* 删除数据库，然后重新setup `diesel database setup`
* 删除数据库，然后重新setup `diesel database reset`
* 创建数据库变更文件 `diesel migration generate create_posts` （`diesel migration generate 变更名`）
    * 将在 `migrations` 目录中创建 一个目录，名为 `${日期}_变更名/`，该目录下包含两个文件
        * `up.sql` 数据库变更更新
        * `down.sql` 数据库变更回滚
    * 注意，变更SQL必须自己写
* 运行所有的 未执行的 migration `diesel migration run` （up.sql）
* 回滚最新的迁移 `diesel migration revert` （down.sql）
* 重新执行最新的迁移 `diesel migration redo` （先执行down.sql、再执行up.sql）
* 更新 `src/schema.rs` 文件  `diesel print-schema`
* Bash自动完成
    * Linux `diesel bash-completion > /etc/bash_completion.d/diesel`
    * OS X
        * `brew install bash-completion  # you may already have this installed`
        * `diesel completions bash > $(brew --prefix)/etc/bash_completion.d/diesel`

### 4、代码编译时执行migration

参考：https://docs.rs/diesel_migrations/1.4.0/diesel_migrations/

```
embed_migrations!("../migrations");
```

不推荐

### 5、配置

https://diesel.rs/guides/configuring-diesel-cli/

主要用于配置 `print_schema`，该子命令用于根据数据库Schema，创建`schema.rs`文件，生成Rust代码。

`diesel.toml`

```toml
# For documentation on how to configure this file,
# see diesel.rs/guides/configuring-diesel-cli

[print_schema]
file = "src/schema.rs"  # schema文件路径
with_docs = true  # 是否创建Docs注释
```

### 6、常用方式

#### （1）场景1使用全部功能

使用以上大部分命令即可

#### （2）仅使用print_schema功能

手动管理数据库Schema，当需要更新表时，调用 `diesel print-schema [table_name]`，然后手动复制到 `src/schema.rs` 文件中（或者重定向）

## 三、增删改查

### 1、更新

#### （1）基本语法

一般用法：通过调用 `diesel::update(source).set(changes)` 配置 `where` 和 `set`
然后通过 `execute`、`get_result`、`get_results` 执行更新（`get_result`、`get_results` 仅 `Postgresql` 可用）。

`diesel::update(target)` 声明如下：

```rust
pub fn update<T: IntoUpdateTarget>(source: T) -> UpdateStatement<T::Table, T::WhereClause>
```

因此 `source` 必须实现 `IntoUpdateTarget`，创建实现 `IntoUpdateTarget` 类型的方式

* 通过 `diesel_cli` 创建 `schema.rs` 文件，所创建的宏展开后的代码
    * `schema::posts::dsl::posts`
    * `schema::posts::dsl::posts.filter(id.eq(1))`
    * `#[derive(Identifiable)]` 注解的类型

`src/models.rs`

```rust

// 用于更新
#[derive(Identifiable, AsChangeset)]
#[table_name="posts"] // 选填 当 struct 名和表名不一致时
#[primary_key(id)] // 选填 当 主键名不是 id时
pub struct PostForUpdate {
    pub id: i64,
    #[column_name = "title"]
    pub title: String,
    pub body: String,
    pub published: bool,
}
```

`src/lib.rs`

```rust
// cargo test
#[cfg(test)]
mod test {
    use super::*;
    use diesel::mysql::Mysql;
    use diesel::debug_query;
    use models::PostForUpdate;


    #[test]
    fn test_update() {
        use schema::posts::dsl::*;
        // 更新全部
        let query = diesel::update(posts).set(published.eq(false));
        println!("{}", debug_query::<Mysql, _>(&query));

        // 更新部分（附带where语句）
        let target = posts.filter(published.eq(true));
        let query = diesel::update(target).set(published.eq(false));
        println!("{}", debug_query::<Mysql, _>(&query));

        // 更新用户传递的对象 （需 `#[derive(Identifiable)]`）
        let post = PostForUpdate {
            id: 1,
            title: "".to_string(),
            body: "".to_string(),
            published: true,
        };
        let query = diesel::update(&post).set(published.eq(false));
        println!("{}", debug_query::<Mysql, _>(&query));

        // 更新过程中自引用
        let query = diesel::update(&post).set(id.eq(id + 1));
        println!("{}", debug_query::<Mysql, _>(&query));

        // 更新多列（使用元组）
        let query = diesel::update(&post).set(
            (
                published.eq(false),
                title.eq("xxx")
            )
        );
        println!("{}", debug_query::<Mysql, _>(&query));

        // 更新多列（使用对象）（需实现 `#[derive(AsChangeset)]`）
        // 注意默认情况下 None 对象表示忽略更新的字段（通过[changeset_options(treat_none_as_null="true")] 更改）
        let query = diesel::update(&post).set(&post);
        println!("{}", debug_query::<Mysql, _>(&query));

        // 执行查询
        let query = diesel::update(&post).set(&post);
        let effect_lines = query.execute(&connection).unwrap();
        println!("{}", effect_lines);
        // let r = query.get_result::<Post>(&connection); // mysql 不支持
        // posts.save_changes(&connection); // mysql 不支持
    }
}
```

执行查询

* `execute` 执行函数，返回影响的行数
* `get_result` 和 `get_results` 执行获取记录（仅 `Postgre`）
* 如果一个类型注解了 `#[derive(Identifiable, AsChangeset)]`，则可以调用 `foo.save_changes(&conn)`（仅 `Postgre`）

### 2、插入

创建 变更 `diesel migration generate create_users`

```sql
-- migrations/2020-03-04-090155_create_users/up.sql
-- Your SQL goes here
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTO_INCREMENT,
  name TEXT NOT NULL,
  hair_color TEXT,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- migrations/2020-03-04-090155_create_users/down.sql
-- This file should undo anything in `up.sql`
DROP TABLE users;
```

运行变更 `diesel migration run`

`src/schema.rs` 文件如下

```rust
// ...
table! {
    users (id) {
        id -> Integer,
        name -> Text,
        hair_color -> Nullable<Text>,
        created_at -> Timestamp,
        updated_at -> Timestamp,
    }
}

allow_tables_to_appear_in_same_query!(
    posts,
    users,
);
```

模型 `src/models.rs`

```rust
#[derive(Insertable)]
#[table_name = "users"]
pub struct UserForm<'a> {
    pub name: &'a str,
    pub hair_color: Option<&'a str>,
}
```

`src/lib.rs`

```rust

// cargo test
#[cfg(test)]
mod test {
    use super::*;
    use diesel::mysql::Mysql;
    use diesel::debug_query;
    use models::{PostForUpdate, Post};

    #[test]
    fn test_insert() {
        use schema::users::dsl::*;
        use models::UserForm;

        let conn = establish_connection();

        // 如果全有默认值，可以以如下语法插入
        let query = diesel::insert_into(users).default_values();
        println!("{}", debug_query::<Mysql, _>(&query));

        // 插入指定值单个
        let query = diesel::insert_into(users)
            .values(name.eq("Sean"));
        println!("{}", debug_query::<Mysql, _>(&query));

        // 插入指定值多个
        let query = diesel::insert_into(users)
            .values((name.eq("Tess"), hair_color.eq("Brown")));
        println!("{}", debug_query::<Mysql, _>(&query));

        // 插入对象
        let user_form = UserForm {
            name: "Sean",
            hair_color: Some("Black"), // 如果是 None 将使用默认值
        };
        let query = diesel::insert_into(users)
            .values(&user_form);
        println!("{}", debug_query::<Mysql, _>(&query));

        // 批量插入1
        let v= vec![name.eq("Sean"), name.eq("Tess")];
        let query = diesel::insert_into(users)
            .values(&v);
        println!("{}", debug_query::<Mysql, _>(&query));

        // 批量插入2
        let v= vec![Some(name.eq("Sean")), None];
        let query = diesel::insert_into(users)
            .values(&v);
        println!("{}", debug_query::<Mysql, _>(&query));

        // 批量插入3
        let v= vec![
                (name.eq("Sean"), hair_color.eq("Black")),
                (name.eq("Tess"), hair_color.eq("Brown")),
            ];
        let query = diesel::insert_into(users)
            .values(&v);
        println!("{}", debug_query::<Mysql, _>(&query));

        // 批量插入4
        let v= vec![
                (name.eq("Sean"), Some(hair_color.eq("Black"))),
                (name.eq("Ruby"), None),
            ];
        let query = diesel::insert_into(users)
            .values(&v);
        println!("{}", debug_query::<Mysql, _>(&query));

        // 批量插入5
        let v= vec![
                user_form,
                UserForm {
                    name: "Sean",
                    hair_color: Some("Black"),
                }
            ];
        let query = diesel::insert_into(users)
            .values(&v);
        println!("{}", debug_query::<Mysql, _>(&query));
    }
}
```

执行查询

* `execute` 执行函数，返回影响的行数
* `get_result` 和 `get_results` 执行获取记录（仅 `Postgre`）
* 如果一个类型注解了 `#[derive(Identifiable, AsChangeset)]`，则可以调用 `foo.save_changes(&conn)`（仅 `Postgre`）

#### （2）RETURNING 子句

仅 `Postgre` 支持 略

#### （3）Upsert （插入或更新）

参见MySQL 参见：https://docs.diesel.rs/diesel/fn.replace_into.html (replace_into)
Postgres 参见：https://docs.diesel.rs/diesel/pg/upsert/fn.on_constraint.html

## 四、Schema 原理探究

```rust
table! {
    users (id) {
        id -> Integer,
        name -> Text,
        hair_color -> Nullable<Text>,
        created_at -> Timestamp,
        updated_at -> Timestamp,
    }
}
```

宏展开后，基本结构如下

```rust
pub mod schema {
    pub mod users {  // 将创建同名的模块
        pub mod dsl {
            // 在 dsl 重新导出了所有的列和表
            pub use super::columns::{id};
            pub use super::columns::{name};
            pub use super::columns::{hair_color};
            pub use super::columns::{created_at};
            pub use super::columns::{updated_at};
            pub use super::table as users;
        }

        pub struct table;

        impl table {
            pub fn star(&self) -> star {
                star
            }
        }

        pub const all_columns: (id, name, hair_color, created_at, updated_at) =
            (id, name, hair_color, created_at, updated_at);

        pub type SqlType = (Integer, Text, Nullable<Text>, Timestamp, Timestamp);

        pub type BoxedQuery<'a, DB, ST = SqlType> = BoxedSelectStatement<'a, ST, table, DB>;

        impl AsQuery for table {
            /* body omitted */
        }

        impl Table for table {
            /* body omitted */
        }

        impl IntoUpdateTarget for table {
            /* body omitted */
        }

        pub mod columns {
            pub struct star;

            pub struct id;
            impl ::diesel::expression::Expression for id {
                type SqlType = Integer;
            }

            pub struct name;
            impl ::diesel::expression::Expression for name {
                type SqlType = Text;
            }

            pub struct hair_color;
            impl ::diesel::expression::Expression for hair_color {
                type SqlType = Nullable<Text>;
            }

            pub struct created_at;
            impl ::diesel::expression::Expression for created_at {
                type SqlType = Timestamp;
            }

            pub struct update_at;
            impl ::diesel::expression::Expression for updated_at {
                type SqlType = Timestamp;
            }
        }

    }
}
```

## 五、Deriving 特质

参见：https://github.com/diesel-rs/diesel/blob/master/guide_drafts/trait_derives.md

本节简单介绍 可以在 结构体上使用的 派生宏 特质

* [`Queryable`](https://github.com/diesel-rs/diesel/blob/master/guide_drafts/trait_derives.md#queryable)
* [`QueryableByName`](https://github.com/diesel-rs/diesel/blob/master/guide_drafts/trait_derives.md#queryablebyname)
* [`Insertable`](https://github.com/diesel-rs/diesel/blob/master/guide_drafts/trait_derives.md#insertable)
* [`Identifiable`](https://github.com/diesel-rs/diesel/blob/master/guide_drafts/trait_derives.md#identifiable)
* [`AsChangeset`](https://github.com/diesel-rs/diesel/blob/master/guide_drafts/trait_derives.md#aschangeset)
* [`Associations`](https://github.com/diesel-rs/diesel/blob/master/guide_drafts/trait_derives.md#associations)

### 1、Queryable

类型安全，使用SQL构建器

* 该派生宏通过Rust类型系统进行匹配，当发生类型失配时，将在**编译期报错**
* 必须通过实现了 `RunQueryDsl` 的对象（也就是 `schema::表名::dsl::表名.load::<类型>` 调用）
* `diesel::sql_query().load::<类型>` 不允许使用
* 结构体字段类型必须 与 Schema 中保持一致，否则编译错误
* 结构体 与 Schema 的绑定映射发生在调用阶段

```rust
// File: src/models.rs

use schema::users; // Brings the users table into scope

#[derive(Queryable)]
pub struct User {
    pub id: i32,
    pub first_name: String,
    pub last_name: String,
    pub email: String,
}

#[derive(Queryable)]
pub struct EmailUser {
    pub id: i32,
    pub email: String,
}

// File: src/main.rs
#[macro_use] extern crate diesel;

use diesel::prelude::*;

fn main() {
    // The following will return all users as a `QueryResult<Vec<User>>`
    let users_result: QueryResult<Vec<User>> = users.load(&db_connection);

    // Here we are getting the value (or error) out of the `QueryResult`
    // A successful value will be of type `Vec<User>`
    let users = users_result.expect("Error loading users");

    // Here, a successful value will be type `Vec<EmailUser>`
    let email_users = users.select((users::id, users::email))
        .load::<EmailUser>(&db_connection)
        .expect("Error loading the email only query");
}
```

### 2、QueryableByName

非类型安全，使用 SQL 字符串查询时使用

* 该派生宏作用用于通过字段名进行类型匹配，匹配发生在运行时，通过名称，报错将发生在运行时
* 使用 `diesel::sql_query().load::<类型>` 时必须添加该 派生宏
* 使用派生宏需要配置各个字段的类型，有两种方式
    * 方式1：通过 引入 Schema 并通过表名配置 `#[table_name="sequences"]`
    * 方式2：为每个字段添加类型注解 `#[sql_type = "Bigint"]`

```rust
// 方式1
use diesel::sql_types::Bigint;

#[derive(QueryableByName)]
// #[table_name="sequences"]
pub struct Sequence {
    #[sql_type = "Bigint"]
    pub id: i64,
}

// 方式2
table! {
    sequences(id) {
        id -> BigInt,
    }
}

// 用于获取id
#[table_name="sequences"]
pub struct Sequence {
    pub id: i64,
}
```

使用

```rust
use self::models::Sequence;

// 获取到id
let generated_id = diesel::sql_query("select LAST_INSERT_ID() as id")
    .load::<Sequence>(conn).expect("get_id_error").first().unwrap().id;
```

### 3、Insertable

用于将实体类型插入数据库

* 基本语法 `#[derive(Insertable)]`
* 必须使用过 `#[table_name="users"]` （users必须导入 `use super::schema::users`） 指明所对应的表
* 字段与数据表列的映射可以使用 `#[column_name = "email"]` 注解

### 4、Identifiable

表示该实体类可以唯一与数据表中的一行映射

* 基本语法 `#[derive(Identifiable)]`，配置
    * `#[table_name="posts"]` 选填 当 struct 名和表名不一致时
    * `#[primary_key(id)]` 选填 当 主键名不是 id 时
    * `#[column_name = "title"]` 选填 注解在 字段上
* 一般在 `diesel::update(&post)` 或者 `diesel::delete(&post)` 使用

### 5、AsChangeset

表示该实体类可以用于update的set的值

* 基本语法 `#[derive(AsChangeset)]`
* 默认情况下 不会更新主键
* 如果主键不是 id，必须使用过 `#[primary_key(your_key)]` 指明
* 默认情况下 `None` 对象表示忽略更新的字段
    * 通过 `#[changeset_options(treat_none_as_null="true")]` 配置 None 字段以 null 更新
    * 另外一种方案可以通过 `Option<Option<T>>` 可以完全处理这个问题
        * `None` 表示忽略
        * `Some(None)` 表示更新为 `NULL`
        * `Some(Some(v))` 表示更新为 `v`

### 6、Associations

参见：https://docs.diesel.rs/diesel/associations/index.html

设置关联关系，不管多对一、一对一、多对多，在实现上都是通过id关联的。diesel通过 父子关系表述，比如 `User` 和 `Post`

* `User` 是父亲 （一对多，一的一方，被参考者）
* `Post` 是孩子 （一对多，多的一方，包含一一方的Id）

因此需要在 孩子一方添加 `#[derive(Associations)]`，并指明其父亲 `[belongs_to(ParentStruct)]`。
默认情况下指向父亲的 id 字段约定为 `parent_id`，可以通过 `#[belongs_to(ParentStruct, foreign_key="my_custom_key")]` 方式自定义。

假设 Schema 格式如下

```rust
// Output of "diesel print-schema"

table! {
    posts (id) {
        id -> Int4,
        user_id -> Int4,
        title -> Varchar,
        content -> Varchar,
    }
}

table! {
    users (id) {
        id -> Int4,
        first_name -> Varchar,
        last_name -> Varchar,
        email -> Nullable<Varchar>,
    }
}
```

使用上

```rust
    // Get the first user
    let issac = users.first::<User>(&db_connection)
        .expect("Couldn't find first user");

    // Get all the posts belonging to the first user
    let issacs_posts = Post::belonging_to(&issac)
        .get_results::<Post>(&db_connection)
        .expect("Couldn't find associated posts");

    // group by 例子
    let users = users::table.load::<User>(&connection)?;
    let posts = Post::belonging_to(&users)
        .load::<Post>(&connection)?
        .grouped_by(&users);
    let data = users.into_iter().zip(posts).collect::<Vec<_>>();

    let expected_data = vec![
        (
            User { id: 1, name: "Sean".into() },
            vec![
                Post { id: 1, user_id: 1, title: "My first post".into() },
                Post { id: 2, user_id: 1, title: "About Rust".into() },
            ],
        ),
        (
            User { id: 2, name: "Tess".into() },
            vec![
                Post { id: 3, user_id: 2, title: "My first post too".into() },
            ],
        ),
    ];

    assert_eq!(expected_data, data);
```

## 六、扩展Diesel的查询生成器

### 1、通过 `sql_function!` 宏自义定函数

* `sql_function!` 用于声明一个SQL函数
* 不支持可变参数，例如concat，这类函数建议使用自定义操作符实现
* 基本使用方式，参见下文例子
    * 该宏需要提供一个 Rust 函数声明，参数和返回值必须是 `use diesel::sql_types::*` 的类型
    * 聚合函数使用 `#[aggregate]`
    * 函数名和 rust 声明的名字不一致建议时使用 `#[sql_name = "concat"]` 指明

声明 `src/lib.rs`

```rust
/// 声明函数
pub mod functions {
    use diesel::sql_types::{Text, BigInt, Foldable};
    sql_function!(
        /// 转换为小写
        fn lower(x: Text) -> Text;
    );
    sql_function!(
        /// 字符串长度
        fn length(x: Text) -> BigInt; // 使用BigInt的原因是字符串最大长度为4G亿
    );
    sql_function!(
        /// 字符串拼接
        #[sql_name = "concat"]
        fn concat2(x: Text, y: Text) -> BigInt;
    );
    sql_function!(
        /// 聚合函数 sum
        #[aggregate]
        #[sql_name = "SUM"]
        fn sum<ST: Foldable>(expr: ST) -> ST::Sum;
    );
    // 无参数SQL函数
    no_arg_sql_function!(last_insert_id, BigInt, "no_arg_sql_function()");

}

pub mod helper_types {
    /// The return type of `length(expr)`
    pub type Lower<Expr> = super::functions::lower::HelperType<Expr>;
    // 其他略
}

pub mod dsl {
    pub use super::functions::*;
    pub use super::helper_types::*;
}
```

使用 `src/lib.rs`

```rust
// cargo test
#[cfg(test)]
mod test {
    use super::*;
    use diesel::mysql::Mysql;
    use diesel::debug_query;
    use models::{PostForUpdate, Post};

    #[test]
    fn test_extend_query_builder() {
        use schema::users::dsl::*;
        use super::dsl::*;

        let conn = establish_connection();

        // 自定义函数
        let query = users.select(lower(name));
        println!("{}", debug_query::<Mysql, _>(&query));

        let query = users.select(concat2(name, "name"));
        // 等价于如下操作符的方式
        // let query = users.select(name.concat("name"));
        println!("{}", debug_query::<Mysql, _>(&query));

        let query = diesel::select(last_insert_id);
        let result = query.get_result::<i64>(&conn).unwrap();
        println!("sql = {}, result = {}", debug_query::<Mysql, _>(&query), result);
    }
}
```

### 2、使用自定义 SQL 扩展 Query DSL

基本步骤

* 创建一个结构体
    * 使用 `#[derive(Debug, Clone, Copy, QueryId)]` 进行扩展
    * 为这个结构体 实现 自有方法
    * 为这个结构体 实现 `QueryFragment`
    * 可选的为这个结构体 实现 `Query`、`RunQueryDsl`
* 创建一个工厂特质，用于用于构造以上的结构体
    * 为需要实现这个功能的特质实例注入这个方法并实现

以实现分页功能为例：

`src/pagination.rs`

```rust
//! 实现分页
//!
//! 仅仅为了演示，应该不需要，因为分页可以通过组合SQL实现
//! 参考 https://github.com/diesel-rs/diesel/blob/v1.3.0/examples/postgres/advanced-blog-cli/src/pagination.rs
use diesel::prelude::*;
use diesel::query_dsl::methods::LoadQuery;
use diesel::query_builder::{QueryFragment, Query, AstPass};
use diesel::mysql::Mysql;
use diesel::sql_types::BigInt;

// 分页特质，用于创建一个Paginated特质
pub trait PaginateForQueryFragment: Sized {
    fn paginate(self, page: i64) -> Paginated<Self>;
}

// 给所有 QueryFragment 类型添加该方法
impl<T> PaginateForQueryFragment for T
where T: QueryFragment<Mysql>{
    /// page: 当前页数
    fn paginate(self, page: i64) -> Paginated<Self> {
        Paginated {
            query: self,
            per_page: 10,
            page,
            is_sub_query: true,
        }
    }
}

// https://docs.diesel.rs/diesel/query_builder/trait.QueryId.html
// QueryID 的作用是：用于实现生成的SQL语句的缓存
// 用于实现分页的结构
#[derive(Debug, Clone, Copy, QueryId)]
pub struct Paginated<T> {
    /// 子查询/表名是什么？
    query: T,
    /// 当前页码
    page: i64,
    /// 每页多行行
    per_page: i64,
    /// query 是否是子查询
    is_sub_query: bool,
}

impl<T> Paginated<T> {
    /// 每页多少条
    pub fn per_page(self, per_page: i64) -> Self {
        Paginated { per_page, ..self }
    }

    /// 实现类似于 load(&conn) 的函数
    /// 获取 <Vec<U>, 总页数>
    pub fn load_and_count_pages<U>(self, conn: &MysqlConnection) -> QueryResult<(Vec<U>, i64)>
    where
        Self: LoadQuery<MysqlConnection, (U, i64)>,
    {
        let per_page = self.per_page;
        let results = self.load::<(U, i64)>(conn)?;
        let total = results.get(0).map(|x| x.1).unwrap_or(0);
        let records = results.into_iter().map(|x| x.0).collect();
        let total_pages = (total as f64 / per_page as f64).ceil() as i64;
        Ok((records, total_pages))
    }
}

// 表示该类型可以生成完整的SQL语句，可以进行查询
impl<T: Query> Query for Paginated<T> {
    type SqlType = (T::SqlType, BigInt);
}

// 添加一系列查询方法如：execute 等
impl<T> RunQueryDsl<MysqlConnection> for Paginated<T> {}

// 核心函数：用于生成SQL
impl<T> QueryFragment<Mysql> for Paginated<T>
where
    T: QueryFragment<Mysql>,
{
    fn walk_ast(&self, mut out: AstPass<Mysql>) -> QueryResult<()> {
        // TODO 优化（使用ID分页）
        out.push_sql("SELECT *, COUNT(*) OVER () FROM ");
        if self.is_sub_query {
            out.push_sql("(");
        }
        self.query.walk_ast(out.reborrow())?;
        if self.is_sub_query {
            out.push_sql(")");
        }
        out.push_sql(" t LIMIT ");
        out.push_bind_param::<BigInt, _>(&self.per_page)?;
        out.push_sql(" OFFSET ");
        let offset = (self.page - 1) * self.per_page;
        out.push_bind_param::<BigInt, _>(&offset)?;
        Ok(())
    }
}

// 以下内容为：为数据源添加分页功能

/// 包装 QuerySource 类型，是之成为 QueryFragment 类型
#[derive(Debug, Clone, Copy, QueryId)]
pub struct QuerySourceToQueryFragment<T> {
    query_source: T,
}

impl<FC, T> QueryFragment<Mysql> for QuerySourceToQueryFragment<T>
where
    FC: QueryFragment<Mysql>,
    T: QuerySource<FromClause=FC>,
{
    fn walk_ast(&self, mut out: AstPass<Mysql>) -> QueryResult<()> {
        self.query_source.from_clause().walk_ast(out.reborrow())?;
        Ok(())
    }
}

// 为 QuerySource 类型添加分页功能
pub trait PaginateForQuerySource: Sized {
    fn paginate(self, page: i64) -> Paginated<QuerySourceToQueryFragment<Self>>;
}

impl<T> PaginateForQuerySource for T
where T: QuerySource {
    /// page: 当前页数
    fn paginate(self, page: i64) -> Paginated<QuerySourceToQueryFragment<Self>> {
        Paginated {
            query: QuerySourceToQueryFragment {query_source: self},
            per_page: 10,
            page,
            is_sub_query: false, // 不是子查询
        }
    }
}
```

使用 `src/lib.rs`

```rust
pub mod pagination;

// cargo test
#[cfg(test)]
mod test {
    use super::*;
    use diesel::mysql::Mysql;
    use diesel::debug_query;
    use models::{PostForUpdate, Post};

    #[test]
    fn test_extend_query_builder() {
        use schema::users::dsl::*;
        use super::dsl::*;

        let conn = establish_connection();

        // 自定义SQL扩展查询DSL
        {
            use schema::posts::dsl::*;
            use super::pagination::{PaginateForQueryFragment, PaginateForQuerySource};

            let query = posts.filter(id.ge(0).and(id.ge(0))).paginate(1).per_page(3);
            println!("{:?}", id);
            let r = query.load_and_count_pages::<Post>(&conn).unwrap();
            println!("sql = {}, {:?}", debug_query::<Mysql, _>(&query), r);

            let query = posts.paginate(1).per_page(3);
            // let query = posts.select(id);
            println!("sql = {}", debug_query::<Mysql, _>(&query));
        }
    }
}
```

### 3、自定义操作符

```rust
// use diesel::mysql::Mysql;
use diesel::sql_types::{BigInt, Integer};
use diesel::expression::{Expression, AsExpression};

// src/expression/operators.rs
// src/expression_methods/text_expression_methods.rs

// diesel_infix_operator!(Add, " @@ ", backend: Mysql);
// 当然最好使用 重载运算符实现参考
// src/sql_types/ops.rs
// src/expression/ops/mod.rs
// src/expression/ops/numeric.rs
// https://docs.diesel.rs/diesel/sql_types/ops/index.html
diesel_infix_operator!(MyBitAnd, " & ", ReturnBasedOnArgs);
diesel_infix_operator!(MyBitOr, " | ", ReturnBasedOnArgs);

pub trait BigIntBitOperatorExtensions: Expression + Sized {

    fn bit_and<T: AsExpression<Self::SqlType>>(self, other: T) -> MyBitAnd<Self, T::Expression> {
        MyBitAnd::new(self, other.as_expression())
    }

    fn bit_or<T: AsExpression<Self::SqlType>>(self, other: T) -> MyBitOr<Self, T::Expression> {
        MyBitOr::new(self, other.as_expression())
    }
}

impl<T: Expression<SqlType=BigInt>> BigIntBitOperatorExtensions for T {
}

pub trait IntegerBitOperatorExtensions: Expression + Sized {

    fn bit_and<T: AsExpression<Self::SqlType>>(self, other: T) -> MyBitAnd<Self, T::Expression> {
        MyBitAnd::new(self, other.as_expression())
    }

    fn bit_or<T: AsExpression<Self::SqlType>>(self, other: T) -> MyBitOr<Self, T::Expression> {
        MyBitOr::new(self, other.as_expression())
    }
}

impl<T: Expression<SqlType=Integer>> IntegerBitOperatorExtensions for T {
}
```

使用 `src/lib.rs`

```rust
pub mod predicates;

// cargo test
#[cfg(test)]
mod test {
    use super::*;
    use diesel::mysql::Mysql;
    use diesel::debug_query;
    use models::{PostForUpdate, Post};

    #[test]
    fn test_extend_query_builder() {
        use schema::users::dsl::*;
        use super::dsl::*;

        let conn = establish_connection();

        // 自定义运算符
        {
            use predicates::*;
            use schema::posts::dsl::*;
            id.bit_and(1);
            let query = posts.filter(id.ge(id.bit_and(1)));
            println!("sql = {}", debug_query::<Mysql, _>(&query));
        }
    }
}
```

### 4、重载运算符

diesel 内部仅实现了 加减乘除 运算符

经试验，在第三方库实现过于复杂，不建议使用，原因如下：

* 孤儿规则的存在
* 相关特质使用了 `type` 语法，因此必须为所有 struct 添加实现，过于繁琐

## 七、r2d2 连接池

更改依赖

```toml
diesel = { version = "1.4", features = ["mysql", "r2d2"] }
```

添加函数

```rust
use diesel::r2d2;

pub fn new_connection_pool() -> r2d2::Pool<r2d2::ConnectionManager<MysqlConnection>> {
    dotenv().ok();

    let database_url = env::var("DATABASE_URL")
        .expect("DATABASE_URL must be set");
    let manager = r2d2::ConnectionManager::<MysqlConnection>::new(database_url);
    let pool = r2d2::Pool::builder()
        .max_size(15)
        .build(manager)
        .expect("Failed to create pool.");
    pool
}
```

测试

```rust

// cargo test
#[cfg(test)]
mod test {
    use super::*;
    use diesel::mysql::Mysql;
    use diesel::debug_query;
    use models::{PostForUpdate, Post};


    #[test]
    fn test_conn_pool(){
        use schema::posts;
        let pool = new_connection_pool();
        let conn = pool.get().unwrap();

        let posts = posts::table.load::<Post>(&conn);

        println!("{:?}", posts);
    }
}

```

## 八、最佳实践

### 1、编程环境

依赖

```toml
diesel = { version = "1.4", features = ["mysql", "r2d2"] }
dotenv = "0.15.0"
```

引入宏、包和函数及预定义函数

```rust
#[macro_use]
extern crate diesel;
extern crate dotenv;

use diesel::prelude::*;
use diesel::mysql::MysqlConnection;
use dotenv::dotenv;
use diesel::r2d2;
use std::env;

// 建立连接
pub fn establish_connection() -> MysqlConnection {
    dotenv().ok();

    // 从数据库中拿到环境变量
    let database_url = env::var("DATABASE_URL")
        .expect("DATABASE_URL must be set");
    // 建连MySQL连接
    MysqlConnection::establish(&database_url)
        .expect(&format!("Error connecting to {}", database_url))
}

pub fn new_connection_pool() -> r2d2::Pool<r2d2::ConnectionManager<MysqlConnection>> {
    dotenv().ok();

    let database_url = env::var("DATABASE_URL")
        .expect("DATABASE_URL must be set");
    let manager = r2d2::ConnectionManager::<MysqlConnection>::new(database_url);
    let pool = r2d2::Pool::builder()
        .max_size(15)
        .build(manager)
        .expect("Failed to create pool.");
    pool
}
```

`Schema` 引入（推荐方式2）

* 方式1：`use schema::表名::dsl::*;`
    * 表名引用 `表名`
    * 列引用 `列名`
* 方式2：`use schema::表名;`
    * 表名引用 `表名::table`
    * 列引用 `表名::columns::列名`

表字段类型引入： `use diesel::sql_types::*`

Debug SQL：`use diesel::debug_query;`

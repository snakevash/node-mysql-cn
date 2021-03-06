# mysql

[![NPM Version][npm-image]][npm-url]
[![NPM Downloads][downloads-image]][downloads-url]
[![Node.js Version][node-version-image]][node-version-url]
[![Linux Build][travis-image]][travis-url]
[![Windows Build][appveyor-image]][appveyor-url]
[![Test Coverage][coveralls-image]][coveralls-url]

## Table of Contents

- [Install](#install)
- [Introduction](#introduction)
- [Contributors](#contributors)
- [Sponsors](#sponsors)
- [Community](#community)
- [Establishing connections](#establishing-connections)
- [链接可选项](#链接可选项)
- [SSL options](#ssl-options)
- [终止链接](#终止链接)
- [连接池](#连接池)
- [池选项](#池选项)
- [池事件](#池事件)
- [关闭池中所有链接](#关闭池中所有链接)
- [池集群](#池集群)
- [池集群选项](#池集群选项)
- [交换用户并且打印链接状态](#交换用户并且打印链接状态)
- [服务器断开](#服务器断开)
- [Performing queries](#performing-queries)
- [Escaping query values](#escaping-query-values)
- [Escaping query identifiers](#escaping-query-identifiers)
- [Preparing Queries](#preparing-queries)
- [Custom format](#custom-format)
- [Getting the id of an inserted row](#getting-the-id-of-an-inserted-row)
- [Getting the number of affected rows](#getting-the-number-of-affected-rows)
- [Getting the number of changed rows](#getting-the-number-of-changed-rows)
- [Getting the connection ID](#getting-the-connection-id)
- [Executing queries in parallel](#executing-queries-in-parallel)
- [Streaming query rows](#streaming-query-rows)
- [Piping results with Streams2](#piping-results-with-streams2)
- [Multiple statement queries](#multiple-statement-queries)
- [Stored procedures](#stored-procedures)
- [Joins with overlapping column names](#joins-with-overlapping-column-names)
- [Transactions](#transactions)
- [Timeouts](#timeouts)
- [Error handling](#error-handling)
- [Exception Safety](#exception-safety)
- [Type casting](#type-casting)
- [Connection Flags](#connection-flags)
- [Debugging and reporting problems](#debugging-and-reporting-problems)
- [Running tests](#running-tests)
- [Todo](#todo)

## Install

```sh
$ npm install mysql
```

For information about the previous 0.9.x releases, visit the [v0.9 branch][].

Sometimes I may also ask you to install the latest version from Github to check
if a bugfix is working. In this case, please do:

```sh
$ npm install felixge/node-mysql
```

[v0.9 branch]: https://github.com/felixge/node-mysql/tree/v0.9

## Introduction

This is a node.js driver for mysql. It is written in JavaScript, does not
require compiling, and is 100% MIT licensed.

Here is an example on how to use it:

```js
var mysql      = require('mysql');
var connection = mysql.createConnection({
  host     : 'localhost',
  user     : 'me',
  password : 'secret'
});

connection.connect();

connection.query('SELECT 1 + 1 AS solution', function(err, rows, fields) {
  if (err) throw err;

  console.log('The solution is: ', rows[0].solution);
});

connection.end();
```

From this example, you can learn the following:

* Every method you invoke on a connection is queued and executed in sequence.
* Closing the connection is done using `end()` which makes sure all remaining
  queries are executed before sending a quit packet to the mysql server.

## Contributors

Thanks goes to the people who have contributed code to this module, see the
[GitHub Contributors page][].

[GitHub Contributors page]: https://github.com/felixge/node-mysql/graphs/contributors

Additionally I'd like to thank the following people:

* [Andrey Hristov][] (Oracle) - for helping me with protocol questions.
* [Ulf Wendel][] (Oracle) - for helping me with protocol questions.

[Ulf Wendel]: http://blog.ulf-wendel.de/
[Andrey Hristov]: http://andrey.hristov.com/

## Sponsors

The following companies have supported this project financially, allowing me to
spend more time on it (ordered by time of contribution):

* [Transloadit](http://transloadit.com) (my startup, we do file uploading &
  video encoding as a service, check it out)
* [Joyent](http://www.joyent.com/)
* [pinkbike.com](http://pinkbike.com/)
* [Holiday Extras](http://www.holidayextras.co.uk/) (they are [hiring](http://join.holidayextras.co.uk/vacancy/software-engineer/))
* [Newscope](http://newscope.com/) (they are [hiring](http://www.newscope.com/stellenangebote))

If you are interested in sponsoring a day or more of my time, please
[get in touch][].

[get in touch]: http://felixge.de/#consulting

## Community

If you'd like to discuss this module, or ask questions about it, please use one
of the following:

* **Mailing list**: https://groups.google.com/forum/#!forum/node-mysql
* **IRC Channel**: #node.js (on freenode.net, I pay attention to any message
  including the term `mysql`)

## Establishing connections

The recommended way to establish a connection is this:

```js
var mysql      = require('mysql');
var connection = mysql.createConnection({
  host     : 'example.org',
  user     : 'bob',
  password : 'secret'
});

connection.connect(function(err) {
  if (err) {
    console.error('error connecting: ' + err.stack);
    return;
  }

  console.log('connected as id ' + connection.threadId);
});
```

However, a connection can also be implicitly established by invoking a query:

```js
var mysql      = require('mysql');
var connection = mysql.createConnection(...);

connection.query('SELECT 1', function(err, rows) {
  // connected! (unless `err` is set)
});
```

Depending on how you like to handle your errors, either method may be
appropriate. Any type of connection error (handshake or network) is considered
a fatal error, see the [Error Handling](#error-handling) section for more
information.

## 链接可选项

当建立一个连接时, 可以有如下选项:

* `host`: 数据库的主机名称. (默认:`localhost`)
* `port`: 链接的端口. (默认: `3306`)
* `localAddress`: TCP链接的源地址. (可选)
* `socketPath`: unix域套接字. 如使用了 `host` 和 `port`, 那么忽略.
* `user`: 用户名称.
* `password`: 密码.
* `database`: 链接的数据库名称 (可选).
* `charset`: 链接的字符编码. 在SQL中被叫做"校对"(像 'utf_general_ci').
  如果SQL中的字符被设置为('utf8mb4'),那么就使用它.(默认: `'UTF8_GENERAL_CI'`)
* `timezone`: 时区. (默认: `'local'`)
* `connectTimeout`: 初始化数据库链接的超时毫秒.(默认: `10000`)
* `stringifyObjects`: 对象字符串化. 浏览[#501](https://github.com/felixge/node-mysql/issues/501). (默认: `'false'`)
* `insecureAuth`: 允许链接MySQL,用不安全的认证. (默认: `false`)
* `typeCast`: 决定是否把列的值转换成原生的JavaScript对象. (默认: `true`)
* `queryFormat`: 自定义查询格式函数. 浏览 [自定义格式](#custom-format).
* `supportBigNumbers`: 当处理超大数值时 (大整数和浮点数), 你应该打开它 (默认: `false`).
* `bigNumberStrings`: 把大数值转换成JavaScript字符串. (默认: `false`).
  只开启 'supportBigNumbers' 会在数值能够准确的翻译成数值对象时才会转换 ([-2^53, +2^53] 区间).
* `dateStrings`: 强制时间格式转化成字符串. (TIMESTAMP, DATETIME, DATE) (默认: `false`)
* `debug`: 打印协议细节到标准输出. (默认: `false`)
* `trace`: 生成栈细节,当产生错误的时候. (默认: `true`)
* `multipleStatements`: 允许多状态查询. 当心注入攻击. (默认: `false`)
* `flags`: 罗列链接标志. 详情浏览[Connection Flags](#connection-flags).
* `ssl`: ssl参数对象或者是ssl配置文件名称.浏览 [SSL options](#ssl-options).

你也可以使用url字符串来替代对象配置. 例如:

```js
var connection = mysql.createConnection('mysql://user:pass@host/db?debug=true&charset=BIG5_CHINESE_CI&timezone=-0700');
```

Note: The query values are first attempted to be parsed as JSON, and if that
fails assumed to be plaintext strings.

### SSL options

The `ssl` option in the connection options takes a string or an object. When given a string,
it uses one of the predefined SSL profiles included. The following profiles are included:

* `"Amazon RDS"`: this profile is for connecting to an Amazon RDS server and contains the
  certificates from https://rds.amazonaws.com/doc/rds-ssl-ca-cert.pem and
  https://s3.amazonaws.com/rds-downloads/rds-combined-ca-bundle.pem

When connecting to other servers, you will need to provide an object of options, in the
same format as [crypto.createCredentials](http://nodejs.org/api/crypto.html#crypto_crypto_createcredentials_details).
Please note the arguments expect a string of the certificate, not a file name to the
certificate. Here is a simple example:

```js
var connection = mysql.createConnection({
  host : 'localhost',
  ssl  : {
    ca : fs.readFileSync(__dirname + '/mysql-ca.crt')
  }
});
```

You can also connect to a MySQL server without properly providing the appropriate
CA to trust. _You should not do this_.

```js
var connection = mysql.createConnection({
  host : 'localhost',
  ssl  : {
    // DO NOT DO THIS
    // set up your ca correctly to trust the connection
    rejectUnauthorized: false
  }
});
```

## 终止链接

有两种方法终止链接. 优雅的方式是调用 'end()' 方法:

```js
connection.end(function(err) {
  // 现在链接被终止
});
```

这会确保之前排队的查询仍然会发送'COM_QUIT'包给MySQL服务器.如果某个错误出现在'COM_QUIT'包之前,
错误会传给回调函数,但是链接还是会被终止.

另一个可选的方法是调用'destroy()'方法来终止链接.这会导致一个当前socket立即终止.
另外　'destroy()' 保证没有回调和方法会被触发.

```js
connection.destroy();
```

不像 `end()` , `destroy()` 没有回调函数.

## 连接池

直接使用连接池.
```js
var mysql = require('mysql');
var pool  = mysql.createPool({
  connectionLimit : 10,
  host            : 'example.org',
  user            : 'bob',
  password        : 'secret'
});

pool.query('SELECT 1 + 1 AS solution', function(err, rows, fields) {
  if (err) throw err;

  console.log('The solution is: ', rows[0].solution);
});
```

连接池可以分享单个连接,或者管理更多连接.

```js
var mysql = require('mysql');
var pool  = mysql.createPool({
  host     : 'example.org',
  user     : 'bob',
  password : 'secret'
});

pool.getConnection(function(err, connection) {
  // connected! (unless `err` is set)
});
```

当你用完一个链接时,只要调用 'connection.release()', 那么连接池会返回到池中,准备复用.

```js
var mysql = require('mysql');
var pool  = mysql.createPool(...);

pool.getConnection(function(err, connection) {
  // 使用链接
  connection.query( 'SELECT something FROM sometable', function(err, rows) {
    // 释放链接
    connection.release();

    // 不要使用链接, 链接已经回归到链接池了.
  });
});
```

如果你想关闭链接并且把它从池中移除, 使用 'connection.destroy()'.
链接池会在需要的时候创建一个新的链接.

链接池总是在需要的时候才创建链接.如果你配置链接池允许有100个,但是你同时只用5个链接,
那么就只有5个链接被创建. 链接池能够尽量复用链接, 从头选取并且存放到最后.

当之前一个链接重新复用,那么一个ping包会尝试检查这个链接是否成功.

## 池选项

池可以接受所有的链接选项. 当创建一个新的链接, 那么它们会被简单的传给链接构造函数.
池可以接受一些额外选项:

* `acquireTimeout`: 链接捕获超时毫秒. 这会有点跟 `connectTimeout` 不同,
  因为捕获超时一般和创建链接不同. (默认: `10000`)
* `waitForConnections`: 决定池在链接用完之后的行为.
  `true`, 那么池会排队等候一个可用的链接.
  `false`, 那么池会立刻回调错误. (默认: `true`)
* `connectionLimit`: 马上创建的连接数. (默认: `10`)
* `queueLimit`: 池请求的最大数值.`0`, 没有限制. (默认: `0`)

## 池事件

### 链接

池会触发 `connection` 事件, 当产生一个新的链接入池的时候.
如果你想要实现会话变量, 你可以监听 `connection`事件

```js
pool.on('connection', function (connection) {
  connection.query('SET SESSION auto_increment_increment=1')
});
```

### 入队

当回调等待一个可用的连接时,池会触发 `enqueue` 事件.

```js
pool.on('enqueue', function () {
  console.log('等待可用链接');
});
```

## 关闭池中所有链接

当你在使用池时,你应该关闭所有池中链接,不然事件循环总是处在激活状态.
关闭所有的池中链接, 应该使用 `end` 方法:

```js
pool.end(function (err) {
  // 关闭所有的链接
});
```

`end` 方法带有一个可选的回调,可以让你立马知道所有的链接已经被关闭.
关闭是优雅的,所有剩下的请求都会被执行完.

**一旦 `pool.end()` 被执行, 那么 `pool.getConnection` 和 其他操作都不会执行**

## 池集群

池集群提供多主机链接. (group & retry & selector)

```js
// 创建
var poolCluster = mysql.createPoolCluster();

// 增加配置
poolCluster.add(config); // 匿名组
poolCluster.add('MASTER', masterConfig);
poolCluster.add('SLAVE1', slave1Config);
poolCluster.add('SLAVE2', slave2Config);

// 移除配置
poolCluster.remove('SLAVE2'); // By nodeId
poolCluster.remove('SLAVE*'); // By target group : SLAVE1-2

// Target Group : ALL(anonymous, MASTER, SLAVE1-2), Selector : round-robin(default)
poolCluster.getConnection(function (err, connection) {});

// Target Group : MASTER, Selector : round-robin
poolCluster.getConnection('MASTER', function (err, connection) {});

// Target Group : SLAVE1-2, Selector : order
// If can't connect to SLAVE1, return SLAVE2. (remove SLAVE1 in the cluster)
poolCluster.on('remove', function (nodeId) {
  console.log('REMOVED NODE : ' + nodeId); // nodeId = SLAVE1 
});

poolCluster.getConnection('SLAVE*', 'ORDER', function (err, connection) {});

// of namespace : of(pattern, selector)
poolCluster.of('*').getConnection(function (err, connection) {});

var pool = poolCluster.of('SLAVE*', 'RANDOM');
pool.getConnection(function (err, connection) {});
pool.getConnection(function (err, connection) {});

// destroy
poolCluster.end();
```

## 池集群选项
* `canRetry`: `true`, `池集群` 会尝试重新连接. (默认: `true`)
* `removeNodeErrorCount`: 如果链接失败, `errorCount` 增加技术, 一旦超过最大值, 那么被移除.(默认: `5`)
* `defaultSelector`: 默认选择器. (默认: `RR`)
  * `RR`: 交替方式. (Round-Robin)
  * `RANDOM`: 随机选取.
  * `ORDER`: 顺序选取.

```js
var clusterConfig = {
  removeNodeErrorCount: 1, // Remove the node immediately when connection fails.
  defaultSelector: 'ORDER'
};

var poolCluster = mysql.createPoolCluster(clusterConfig);
```

## 交换用户并且打印链接状态

MySQL提供一个交换用户命令可以在同一个socket中交换用户:

```js
connection.changeUser({user : 'john'}, function(err) {
  if (err) throw err;
});
```

这个特性可选的配置:

* `user`: 新用户名称 (默认之前的名称).
* `password`: 密码 (默认之前的名称).
* `charset`: 编码 (默认之前的名称).
* `database`: 数据库名称 (默认之前的名称).

这个功能会重置链接的状态(变量,事务 等等).

发生会错误会当做链接错误.

## 服务器断开

当网络发生问题时会断开服务器,或者服务重启之类的.
所有这些都会发生链接错误,并且抛出 `err.code='PROTOCOL_CONNECTION_LOST'`. 浏览[错误处理](#错误处理).

重新链接是断开旧的重建新的.一旦终止,一个存在的链接将不可重连.

在池中,断开的链接会给新的连接腾出位置.

## Performing queries

In the MySQL library library, the most basic way to perform a query is to call
the `.query()` method on an object (like on a `Connection`, `Pool`, `PoolNamespace`
or other similar objects).

The simplest form on query comes as `.query(sqlString, callback)`, where a string
of a MySQL query is the first argument and the second is a callback:

```js
connection.query('SELECT * FROM `books` WHERE `author` = "David"', function (error, results, fields) {
  // error will be an Error if one occurred during the query
  // results will contain the results of the query
  // fields will contain information about the returned results fields (if any)
});
```

The second form `.query(sqlString, parameters, callback)` comes when using
placeholders (see [escaping query values](#escaping-query-values)):

```js
connection.query('SELECT * FROM `books` WHERE `author` = ?', ['David'], function (error, results, fields) {
  // error will be an Error if one occurred during the query
  // results will contain the results of the query
  // fields will contain information about the returned results fields (if any)
});
```

The third form `.query(options, callback)` comes when using various advanced
options on the query, like [escaping query values](#escaping-query-values),
[joins with overlapping column names](#joins-with-overlapping-column-names),
[timeouts](#timeout), and [type casting](#type-casting).

```js
connection.query({
  sql: 'SELECT * FROM `books` WHERE `author` = ?',
  timeout: 40000, // 40s
  values: ['David']
}, function (error, results, fields) {
  // error will be an Error if one occurred during the query
  // results will contain the results of the query
  // fields will contain information about the returned results fields (if any)
});
```

## Escaping query values

In order to avoid SQL Injection attacks, you should always escape any user
provided data before using it inside a SQL query. You can do so using the
`mysql.escape()`, `connection.escape()` or `pool.escape()` methods:

```js
var userId = 'some user provided value';
var sql    = 'SELECT * FROM users WHERE id = ' + connection.escape(userId);
connection.query(sql, function(err, results) {
  // ...
});
```

Alternatively, you can use `?` characters as placeholders for values you would
like to have escaped like this:

```js
connection.query('SELECT * FROM users WHERE id = ?', [userId], function(err, results) {
  // ...
});
```

This looks similar to prepared statements in MySQL, however it really just uses
the same `connection.escape()` method internally.

**Caution** This also differs from prepared statements in that all `?` are
replaced, even those contained in comments and strings.

Different value types are escaped differently, here is how:

* Numbers are left untouched
* Booleans are converted to `true` / `false`
* Date objects are converted to `'YYYY-mm-dd HH:ii:ss'` strings
* Buffers are converted to hex strings, e.g. `X'0fa5'`
* Strings are safely escaped
* Arrays are turned into list, e.g. `['a', 'b']` turns into `'a', 'b'`
* Nested arrays are turned into grouped lists (for bulk inserts), e.g. `[['a',
  'b'], ['c', 'd']]` turns into `('a', 'b'), ('c', 'd')`
* Objects are turned into `key = 'val'` pairs for each enumerable property on
  the object. If the property's value is a function, it is skipped; if the
  property's value is an object, toString() is called on it and the returned
  value is used.
* `undefined` / `null` are converted to `NULL`
* `NaN` / `Infinity` are left as-is. MySQL does not support these, and trying
  to insert them as values will trigger MySQL errors until they implement
  support.

If you paid attention, you may have noticed that this escaping allows you
to do neat things like this:

```js
var post  = {id: 1, title: 'Hello MySQL'};
var query = connection.query('INSERT INTO posts SET ?', post, function(err, result) {
  // Neat!
});
console.log(query.sql); // INSERT INTO posts SET `id` = 1, `title` = 'Hello MySQL'

```

If you feel the need to escape queries by yourself, you can also use the escaping
function directly:

```js
var query = "SELECT * FROM posts WHERE title=" + mysql.escape("Hello MySQL");

console.log(query); // SELECT * FROM posts WHERE title='Hello MySQL'
```

## Escaping query identifiers

If you can't trust an SQL identifier (database / table / column name) because it is
provided by a user, you should escape it with `mysql.escapeId(identifier)`,
`connection.escapeId(identifier)` or `pool.escapeId(identifier)` like this:

```js
var sorter = 'date';
var sql    = 'SELECT * FROM posts ORDER BY ' + connection.escapeId(sorter);
connection.query(sql, function(err, results) {
  // ...
});
```

It also supports adding qualified identifiers. It will escape both parts.

```js
var sorter = 'date';
var sql    = 'SELECT * FROM posts ORDER BY ' + connection.escapeId('posts.' + sorter);
connection.query(sql, function(err, results) {
  // ...
});
```

Alternatively, you can use `??` characters as placeholders for identifiers you would
like to have escaped like this:

```js
var userId = 1;
var columns = ['username', 'email'];
var query = connection.query('SELECT ?? FROM ?? WHERE id = ?', [columns, 'users', userId], function(err, results) {
  // ...
});

console.log(query.sql); // SELECT `username`, `email` FROM `users` WHERE id = 1
```
**Please note that this last character sequence is experimental and syntax might change**

When you pass an Object to `.escape()` or `.query()`, `.escapeId()` is used to avoid SQL injection in object keys.

### Preparing Queries

You can use mysql.format to prepare a query with multiple insertion points, utilizing the proper escaping for ids and values. A simple example of this follows:

```js
var sql = "SELECT * FROM ?? WHERE ?? = ?";
var inserts = ['users', 'id', userId];
sql = mysql.format(sql, inserts);
```

Following this you then have a valid, escaped query that you can then send to the database safely. This is useful if you are looking to prepare the query before actually sending it to the database. As mysql.format is exposed from SqlString.format you also have the option (but are not required) to pass in stringifyObject and timezone, allowing you provide a custom means of turning objects into strings, as well as a location-specific/timezone-aware Date.

### Custom format

If you prefer to have another type of query escape format, there's a connection configuration option you can use to define a custom format function. You can access the connection object if you want to use the built-in `.escape()` or any other connection function.

Here's an example of how to implement another format:

```js
connection.config.queryFormat = function (query, values) {
  if (!values) return query;
  return query.replace(/\:(\w+)/g, function (txt, key) {
    if (values.hasOwnProperty(key)) {
      return this.escape(values[key]);
    }
    return txt;
  }.bind(this));
};

connection.query("UPDATE posts SET title = :title", { title: "Hello MySQL" });
```

## Getting the id of an inserted row

If you are inserting a row into a table with an auto increment primary key, you
can retrieve the insert id like this:

```js
connection.query('INSERT INTO posts SET ?', {title: 'test'}, function(err, result) {
  if (err) throw err;

  console.log(result.insertId);
});
```

When dealing with big numbers (above JavaScript Number precision limit), you should
consider enabling `supportBigNumbers` option to be able to read the insert id as a
string, otherwise it will throw.

This option is also required when fetching big numbers from the database, otherwise
you will get values rounded to hundreds or thousands due to the precision limit.

## Getting the number of affected rows

You can get the number of affected rows from an insert, update or delete statement.

```js
connection.query('DELETE FROM posts WHERE title = "wrong"', function (err, result) {
  if (err) throw err;

  console.log('deleted ' + result.affectedRows + ' rows');
})
```

## Getting the number of changed rows

You can get the number of changed rows from an update statement.

"changedRows" differs from "affectedRows" in that it does not count updated rows
whose values were not changed.

```js
connection.query('UPDATE posts SET ...', function (err, result) {
  if (err) throw err;

  console.log('changed ' + result.changedRows + ' rows');
})
```

## Getting the connection ID

You can get the MySQL connection ID ("thread ID") of a given connection using the `threadId`
property.

```js
connection.connect(function(err) {
  if (err) throw err;
  console.log('connected as id ' + connection.threadId);
});
```

## Executing queries in parallel

The MySQL protocol is sequential, this means that you need multiple connections
to execute queries in parallel. You can use a Pool to manage connections, one
simple approach is to create one connection per incoming http request.

## Streaming query rows

Sometimes you may want to select large quantities of rows and process each of
them as they are received. This can be done like this:

```js
var query = connection.query('SELECT * FROM posts');
query
  .on('error', function(err) {
    // Handle error, an 'end' event will be emitted after this as well
  })
  .on('fields', function(fields) {
    // the field packets for the rows to follow
  })
  .on('result', function(row) {
    // Pausing the connnection is useful if your processing involves I/O
    connection.pause();

    processRow(row, function() {
      connection.resume();
    });
  })
  .on('end', function() {
    // all rows have been received
  });
```

Please note a few things about the example above:

* Usually you will want to receive a certain amount of rows before starting to
  throttle the connection using `pause()`. This number will depend on the
  amount and size of your rows.
* `pause()` / `resume()` operate on the underlying socket and parser. You are
  guaranteed that no more `'result'` events will fire after calling `pause()`.
* You MUST NOT provide a callback to the `query()` method when streaming rows.
* The `'result'` event will fire for both rows as well as OK packets
  confirming the success of a INSERT/UPDATE query.
* It is very important not to leave the result paused too long, or you may
  encounter `Error: Connection lost: The server closed the connection.`
  The time limit for this is determined by the
  [net_write_timeout setting](https://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html#sysvar_net_write_timeout)
  on your MySQL server.

Additionally you may be interested to know that it is currently not possible to
stream individual row columns, they will always be buffered up entirely. If you
have a good use case for streaming large fields to and from MySQL, I'd love to
get your thoughts and contributions on this.

### Piping results with Streams2

The query object provides a convenience method `.stream([options])` that wraps
query events into a [Readable](http://nodejs.org/api/stream.html#stream_class_stream_readable)
Streams2[Streams2](http://blog.nodejs.org/2012/12/20/streams2/) object. This
stream can easily be piped downstream and provides automatic pause/resume,
based on downstream congestion and the optional `highWaterMark`. The
`objectMode` parameter of the stream is set to `true` and cannot be changed
(if you need a byte stream, you will need to use a transform stream, like
[objstream](https://www.npmjs.com/package/objstream) for example).

For example, piping query results into another stream (with a max buffer of 5
objects) is simply:

```js
connection.query('SELECT * FROM posts')
  .stream({highWaterMark: 5})
  .pipe(...);
```

## Multiple statement queries

Support for multiple statements is disabled for security reasons (it allows for
SQL injection attacks if values are not properly escaped). To use this feature
you have to enable it for your connection:

```js
var connection = mysql.createConnection({multipleStatements: true});
```

Once enabled, you can execute multiple statement queries like any other query:

```js
connection.query('SELECT 1; SELECT 2', function(err, results) {
  if (err) throw err;

  // `results` is an array with one element for every statement in the query:
  console.log(results[0]); // [{1: 1}]
  console.log(results[1]); // [{2: 2}]
});
```

Additionally you can also stream the results of multiple statement queries:

```js
var query = connection.query('SELECT 1; SELECT 2');

query
  .on('fields', function(fields, index) {
    // the fields for the result rows that follow
  })
  .on('result', function(row, index) {
    // index refers to the statement this result belongs to (starts at 0)
  });
```

If one of the statements in your query causes an error, the resulting Error
object contains a `err.index` property which tells you which statement caused
it. MySQL will also stop executing any remaining statements when an error
occurs.

Please note that the interface for streaming multiple statement queries is
experimental and I am looking forward to feedback on it.

## Stored procedures

You can call stored procedures from your queries as with any other mysql driver.
If the stored procedure produces several result sets, they are exposed to you
the same way as the results for multiple statement queries.

## Joins with overlapping column names

When executing joins, you are likely to get result sets with overlapping column
names.

By default, node-mysql will overwrite colliding column names in the
order the columns are received from MySQL, causing some of the received values
to be unavailable.

However, you can also specify that you want your columns to be nested below
the table name like this:

```js
var options = {sql: '...', nestTables: true};
connection.query(options, function(err, results) {
  /* results will be an array like this now:
  [{
    table1: {
      fieldA: '...',
      fieldB: '...',
    },
    table2: {
      fieldA: '...',
      fieldB: '...',
    },
  }, ...]
  */
});
```

Or use a string separator to have your results merged.

```js
var options = {sql: '...', nestTables: '_'};
connection.query(options, function(err, results) {
  /* results will be an array like this now:
  [{
    table1_fieldA: '...',
    table1_fieldB: '...',
    table2_fieldA: '...',
    table2_fieldB: '...',
  }, ...]
  */
});
```

## Transactions

Simple transaction support is available at the connection level:

```js
connection.beginTransaction(function(err) {
  if (err) { throw err; }
  connection.query('INSERT INTO posts SET title=?', title, function(err, result) {
    if (err) { 
      connection.rollback(function() {
        throw err;
      });
    }

	var log = 'Post ' + result.insertId + ' added';

	connection.query('INSERT INTO log SET data=?', log, function(err, result) {
	  if (err) { 
        connection.rollback(function() {
          throw err;
        });
      }  
	  connection.commit(function(err) {
	    if (err) { 
          connection.rollback(function() {
            throw err;
          });
        }
	    console.log('success!');
	  });
    });
  });
});
```
Please note that beginTransaction(), commit() and rollback() are simply convenience
functions that execute the START TRANSACTION, COMMIT, and ROLLBACK commands respectively.
It is important to understand that many commands in MySQL can cause an implicit commit,
as described [in the MySQL documentation](http://dev.mysql.com/doc/refman/5.5/en/implicit-commit.html)

## Ping

A ping packet can be sent over a connection using the `connection.ping` method. This
mehtod will send a ping packet to the server and when the server responds, the callback
will fire. If an error occurred, the callback will fire with an error argument.

```js
connection.ping(function (err) {
  if (err) throw err;
  console.log('Server responded to ping');
})
```

## Timeouts

Every operation takes an optional inactivity timeout option. This allows you to
specify appropriate timeouts for operations. It is important to note that these
timeouts are not part of the MySQL protocol, and rather timeout operations through
the client. This means that when a timeout is reached, the connection it occurred
on will be destroyed and no further operations can be performed.

```js
// Kill query after 60s
connection.query({sql: 'SELECT COUNT(*) AS count FROM big_table', timeout: 60000}, function (err, rows) {
  if (err && err.code === 'PROTOCOL_SEQUENCE_TIMEOUT') {
    throw new Error('too long to count table rows!');
  }

  if (err) {
    throw err;
  }

  console.log(rows[0].count + ' rows');
});
```

## Error handling

This module comes with a consistent approach to error handling that you should
review carefully in order to write solid applications.

All errors created by this module are instances of the JavaScript [Error][]
object. Additionally they come with two properties:

* `err.code`: Either a [MySQL server error][] (e.g.
  `'ER_ACCESS_DENIED_ERROR'`), a node.js error (e.g. `'ECONNREFUSED'`) or an
  internal error (e.g.  `'PROTOCOL_CONNECTION_LOST'`).
* `err.fatal`: Boolean, indicating if this error is terminal to the connection
  object.

[Error]: https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Error
[MySQL server error]: http://dev.mysql.com/doc/refman/5.5/en/error-messages-server.html

Fatal errors are propagated to *all* pending callbacks. In the example below, a
fatal error is triggered by trying to connect to an invalid port. Therefore the
error object is propagated to both pending callbacks:

```js
var connection = require('mysql').createConnection({
  port: 84943, // WRONG PORT
});

connection.connect(function(err) {
  console.log(err.code); // 'ECONNREFUSED'
  console.log(err.fatal); // true
});

connection.query('SELECT 1', function(err) {
  console.log(err.code); // 'ECONNREFUSED'
  console.log(err.fatal); // true
});
```

Normal errors however are only delegated to the callback they belong to.  So in
the example below, only the first callback receives an error, the second query
works as expected:

```js
connection.query('USE name_of_db_that_does_not_exist', function(err, rows) {
  console.log(err.code); // 'ER_BAD_DB_ERROR'
});

connection.query('SELECT 1', function(err, rows) {
  console.log(err); // null
  console.log(rows.length); // 1
});
```

Last but not least: If a fatal errors occurs and there are no pending
callbacks, or a normal error occurs which has no callback belonging to it, the
error is emitted as an `'error'` event on the connection object. This is
demonstrated in the example below:

```js
connection.on('error', function(err) {
  console.log(err.code); // 'ER_BAD_DB_ERROR'
});

connection.query('USE name_of_db_that_does_not_exist');
```

Note: `'error'` are special in node. If they occur without an attached
listener, a stack trace is printed and your process is killed.

**tl;dr:** This module does not want you to deal with silent failures. You
should always provide callbacks to your method calls. If you want to ignore
this advice and suppress unhandled errors, you can do this:

```js
// I am Chuck Norris:
connection.on('error', function() {});
```

## Exception Safety

This module is exception safe. That means you can continue to use it, even if
one of your callback functions throws an error which you're catching using
'uncaughtException' or a domain.

## Type casting

For your convenience, this driver will cast mysql types into native JavaScript
types by default. The following mappings exist:

### Number

* TINYINT
* SMALLINT
* INT
* MEDIUMINT
* YEAR
* FLOAT
* DOUBLE

### Date

* TIMESTAMP
* DATE
* DATETIME

### Buffer

* TINYBLOB
* MEDIUMBLOB
* LONGBLOB
* BLOB
* BINARY
* VARBINARY
* BIT (last byte will be filled with 0 bits as necessary)

### String

* CHAR
* VARCHAR
* TINYTEXT
* MEDIUMTEXT
* LONGTEXT
* TEXT
* ENUM
* SET
* DECIMAL (may exceed float precision)
* BIGINT (may exceed float precision)
* TIME (could be mapped to Date, but what date would be set?)
* GEOMETRY (never used those, get in touch if you do)

It is not recommended (and may go away / change in the future) to disable type
casting, but you can currently do so on either the connection:

```js
var connection = require('mysql').createConnection({typeCast: false});
```

Or on the query level:

```js
var options = {sql: '...', typeCast: false};
var query = connection.query(options, function(err, results) {

});
```

You can also pass a function and handle type casting yourself. You're given some
column information like database, table and name and also type and length. If you
just want to apply a custom type casting to a specific type you can do it and then
fallback to the default. Here's an example of converting `TINYINT(1)` to boolean:

```js
connection.query({
  sql: '...',
  typeCast: function (field, next) {
    if (field.type == 'TINY' && field.length == 1) {
      return (field.string() == '1'); // 1 = true, 0 = false
    }
    return next();
  }
});
```
__WARNING: YOU MUST INVOKE the parser using one of these three field functions in your custom typeCast callback. They can only be called once.( see #539 for discussion)__

```
field.string()
field.buffer()
field.geometry()
```
are aliases for
```
parser.parseLengthCodedString()
parser.parseLengthCodedBuffer()
parser.parseGeometryValue()
```
__You can find which field function you need to use by looking at: [RowDataPacket.prototype._typeCast](https://github.com/felixge/node-mysql/blob/master/lib/protocol/packets/RowDataPacket.js#L41)__


## Connection Flags

If, for any reason, you would like to change the default connection flags, you
can use the connection option `flags`. Pass a string with a comma separated list
of items to add to the default flags. If you don't want a default flag to be used
prepend the flag with a minus sign. To add a flag that is not in the default list,
just write the flag name, or prefix it with a plus (case insensitive).

**Please note that some available flags that are not not supported (e.g.: Compression),
are still not allowed to be specified.**

### Example

The next example blacklists FOUND_ROWS flag from default connection flags.

```js
var connection = mysql.createConnection("mysql://localhost/test?flags=-FOUND_ROWS");
```

### Default Flags

The following flags are sent by default on a new connection:

- `CONNECT_WITH_DB` - Ability to specify the database on connection.
- `FOUND_ROWS` - Send the found rows instead of the affected rows as `affectedRows`.
- `IGNORE_SIGPIPE` - Old; no effect.
- `IGNORE_SPACE` - Let the parser ignore spaces before the `(` in queries.
- `LOCAL_FILES` - Can use `LOAD DATA LOCAL`.
- `LONG_FLAG`
- `LONG_PASSWORD` - Use the improved version of Old Password Authentication.
- `MULTI_RESULTS` - Can handle multiple resultsets for COM_QUERY.
- `ODBC` Old; no effect.
- `PROTOCOL_41` - Uses the 4.1 protocol.
- `PS_MULTI_RESULTS` - Can handle multiple resultsets for COM_STMT_EXECUTE.
- `RESERVED` - Old flag for the 4.1 protocol.
- `SECURE_CONNECTION` - Support native 4.1 authentication.
- `TRANSACTIONS` - Asks for the transaction status flags.

In addition, the following flag will be sent if the option `multipleStatements`
is set to `true`:

- `MULTI_STATEMENTS` - The client may send multiple statement per query or
  statement prepare.

### Other Available Flags

There are other flags available. They may or may not function, but are still
available to specify.

- COMPRESS
- INTERACTIVE
- NO_SCHEMA
- PLUGIN_AUTH
- REMEMBER_OPTIONS
- SSL
- SSL_VERIFY_SERVER_CERT

## Debugging and reporting problems

If you are running into problems, one thing that may help is enabling the
`debug` mode for the connection:

```js
var connection = mysql.createConnection({debug: true});
```

This will print all incoming and outgoing packets on stdout. You can also restrict debugging to
packet types by passing an array of types to debug:

```js
var connection = mysql.createConnection({debug: ['ComQueryPacket', 'RowDataPacket']});
```

to restrict debugging to the query and data packets.

If that does not help, feel free to open a GitHub issue. A good GitHub issue
will have:

* The minimal amount of code required to reproduce the problem (if possible)
* As much debugging output and information about your environment (mysql
  version, node version, os, etc.) as you can gather.

## Running tests

The test suite is split into two parts: unit tests and integration tests.
The unit tests run on any machine while the integration tests require a
MySQL server instance to be setup.

### Running unit tests

```sh
$ FILTER=unit npm test
```

### Running integration tests

Set the environment variables `MYSQL_DATABASE`, `MYSQL_HOST`, `MYSQL_PORT`,
`MYSQL_USER` and `MYSQL_PASSWORD`. Then run `npm test`.

For example, if you have an installation of mysql running on localhost:3306
and no password set for the `root` user, run:

```sh
$ mysql -u root -e "CREATE DATABASE IF NOT EXISTS node_mysql_test"
$ MYSQL_HOST=localhost MYSQL_PORT=3306 MYSQL_DATABASE=node_mysql_test MYSQL_USER=root MYSQL_PASSWORD= FILTER=integration npm test
```

## Todo

* Prepared statements
* Support for encodings other than UTF-8 / ASCII

[npm-image]: https://img.shields.io/npm/v/mysql.svg
[npm-url]: https://npmjs.org/package/mysql
[node-version-image]: http://img.shields.io/node/v/mysql.svg
[node-version-url]: http://nodejs.org/download/
[travis-image]: https://img.shields.io/travis/felixge/node-mysql/master.svg?label=linux
[travis-url]: https://travis-ci.org/felixge/node-mysql
[appveyor-image]: https://img.shields.io/appveyor/ci/dougwilson/node-mysql/master.svg?label=windows
[appveyor-url]: https://ci.appveyor.com/project/dougwilson/node-mysql
[coveralls-image]: https://img.shields.io/coveralls/felixge/node-mysql/master.svg
[coveralls-url]: https://coveralls.io/r/felixge/node-mysql?branch=master
[downloads-image]: https://img.shields.io/npm/dm/mysql.svg
[downloads-url]: https://npmjs.org/package/mysql

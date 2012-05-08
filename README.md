# PHP MVC 框架和应用

## 目录说明

* config_dev: 开发环境配置
* config_prd: 生产环境配置
* library: 核心库和类
* static: 静态资源，例如 js,css,图片等
* skins: 页面模板
* web/*/index.php: 各网站入口

## 应用程序示例

### 1. Model: /library/class/Sp/Web/Catalog/Model/Item.php
~~~

namespace Sp\Web\Catalog;

/**
* Model_Item
*/
class Model_Item extends \Model
{
	
	protected static $_db_name = 'sp.catalog';
	protected static $_table_name = 'catalog_items';
	protected static $_primary_key = array('id');
	
	// TODO: 添加业务逻辑接口
	
}

~~~

### 2. Controller: /library/class/Sp/Web/Catalog/Controller/Item.php
~~~

namespace Sp\Web\Catalog;

/**
* Controller_Item
*/
class Controller_Item extends \Sp\Web\Controller
{
	public function action_index()
	{
		# code...
		//echo 'Welcome ', time(), PHP_EOL;
		return array(302, '/home/');
	}
	
	public function action_info($id = NULL)
	{
		# code...
		//var_dump('id: ', $id);
		if ($id) {
			$item = Model_Item::find($id);
		}
		else {
			//$item = Model_Item::find('first');
			return 404;
		}
		//var_dump($item);
		
		$data = array('item', $item);
		return array('catalog/item_info.tpl', $data);
	}
}

~~~

### 3. View: /skins/default/catalog/item_info.tpl
~~~
html:
{$item.id}
~~~


### 4. 入口： /web/catalog/index.php
~~~

include_once 'init.php';

defined('DOC_ROOT') || define('DOC_ROOT', __DIR__.DS);

Dispatcher::dispatch(array(
	'namespace' => '\Sp\Web\Catalog',
	'default_controller' => 'home',
	'default_action' => 'index'
));

// or TODO: 修改 Dispatcher 定义，允许不使用namespace, 并添加自定义类位置

Loader::import(WEB_ROOT . 'appclass');
Dispatcher::dispatch(array(
	'default_controller' => 'home',
	'default_action' => 'index'
));

~~~

### 5. Nginx 虚拟主机配置: /config_dev/nginx/host.12.catalog.test.conf
~~~

server {
	listen		  80;
	server_name	  catalog.test;
	root   /sproot/web/catalog;
	index  index.htm index.html index.php;

	include global/restrictions.conf;

	location / {
		try_files $uri $uri/ /index.php?$args;
	}

	location ~ \.php$ {
		fastcgi_pass   php;
		fastcgi_index  index.php;
		include		fastcgi.conf;
	}

}


~~~

### 6. URL规则

URL: 

> http:// catalog.test /item/info/1234

调度器将查找类 `Controller_Item` 的 `action_info` 方法，并将后续的1234作为参数传入，因此调用过程将映射为：

> index.php -> Dispatcher::dispatch -> appclass/Controller/Item::action_info ('1234')


URL:

> http:// webapp.test /account/1234/edit

由于不存在名为1234的action方法，会查找`__call`方法，如果存在，将映射为：

> index.php -> Dispatcher::dispatch -> appclass/Controller/Account::__call(1234, array('edit'))




## 多域配置支持
* 应用在init阶段会判断是否传入域参数（HTTP_MDOMAIN），然后会指示Loader只加载指定域的配置
* Client 端在http request header 中 添加 **MDomain** 值
* 产生新Domain配置的步骤：

~~~
cd config_dev （生产环境 config_prd）
mkdir _newdomain （创建一个下划线开头的目录）
cp *.conf.php *.ini *.yml _newdomain/
~~~


## 数据库操作
### 配置文件

	$ cat da.conf.php

~~~
return array(
	'default' => 'sp.cms.read',
	'sp' => array(
		'passport' => array(
			'dsn' => 'mysql:host=localhost;dbname=passport',
			'charset' => 'utf8',
			'username' => 'dbuser',
			'password' => 'secretpassword'
		)
		,
		'cms' => array(
			'r' => array(
				'dsn' => 'mysql:host=localhost;dbname=cms',
				'username' => 'dbuser',
				'password' => 'secretpassword'
			)
			,
			'w' => array(
				'dsn' => 'mysql:host=localhost;dbname=cms',
				'username' => 'dbuser',
				'password' => 'secretpassword'
			)
		)
	)
	
);
~~~
### 数据操作的两种方式

1. 使用对象链式操作：

~~~
$rows = Da_Wrapper::select()
	->db('sp.passport')
	->table('users')
	->columns('id,name,email,created,status')
	->where('status', '=', 2)
	->limit(10, 0)
	->getAll();

$ret = Da_Wrapper::update()
	->table('sp.passport.users')
	->data(array('name'=>'newname'))
	->where('id', 12)
	->execute();

$new_id = Da_Wrapper::insert()
	->table('sp.passport.users')
	->data(array('name'=>'newname','email'=>'newemail'))
	->execute();

$ret = Da_Wrapper::delete()
	->table('sp.passport.users')
	->where('id',12)
	->execute();
~~~

> 其中db()方法可以省略，如省略会找到 da.conf 中的 default 项目，或者在table()方法中指定，table()的参数格式： `ns.db[.node][.schema].table`

2. 直接执行SQL

~~~
$user_data = Da_Wrapper::getRow('sp.passport.users', 'SELECT * from users WHERE id = ?', array(12));
~~~

Eol.

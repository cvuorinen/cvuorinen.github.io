---
title: Model testing using SQLite in-memory database with Zend Framework
categories:
  - Programming
tags:
  - PHP
  - PHPUnit
  - SQLite
  - Testing
  - Zend Framework
---

In this post I am going to describe the method we set up with a few coworkers for testing models in Zend Framework projects. We are using fixtures to ensure our database is in a consistent state for the tests and we wanted to run the tests on the same machine we are using for development, but usually the database you use for development is full of data that you don't want to lose. We also wanted to be able to run the tests easily on any new dev machine, test server, continous integration server etc.

<!--more-->

So we decided to use a completely separate SQLite database for testing. SQLite seemed ideal since it would not need anything installed or configured, it should just work on any machine. There is also the very handy [in-memory database](http://www.sqlite.org/inmemorydb.html) that is great for this use case, since the database is only needed while running the tests, it's faster and it does not require any file/directory permissions to be modified in order to work.

## Configuration

The SQLite in-memory database is quite simple to set up, you only need to use the keyword **:memory:** as the filename. This creates a temporary database in memory, that requires no file handles and that will be destroyed when the connection is closed.

Zend Framework resource plugins simplify database connection instantiation, but we cannot use the standard db resource plugin since it is already used by the main database connection. Luckily there is also a [multidb resource plugin](http://framework.zend.com/manual/1.10/en/zend.application.available-resources.html#zend.application.available-resources.multidb) that is ideal for this situation, as it allows to configure multiple named connections that can be retrieved as needed. So here is what our database configuration looks like:

```
; Database connetion
resources.db.adapter = "Pdo_Mysql"
resources.db.params.charset = "utf8"
resources.db.params.host = "localhost"
resources.db.params.username = "root"
resources.db.params.password = ""
resources.db.params.dbname = "dbname"

; Database for unit testing
resources.multidb.testdb.adapter = "PDO_SQLITE"
resources.multidb.testdb.dbname = ":memory:"
resources.multidb.testdb.testschema.file = APPLICATION_PATH "/../tests/data/sqlite_schema.sql"
```

## Setup

To set up the database adapter and bootstrap our application, we use an abstract DatabaseTestCase class that extends *PHPUnit_Extensions_Database_TestCase*. In the class's *setUp* method, we bootstrap our Zend Framework application (not the whole MVC stack, only what is actually needed) and the retrieve the database connection and schema filename. In *getConnection* method we create the actual database connection, register the db adapter with Zend_Db_Table as default adapter (since we are using Zend_Db_Table with our models) and execute the schema file to set up the database with our table structure. Here is the actual code:

```php?start_inline=1
abstract class My_Test_PHPUnit_DatabaseTestCase extends PHPUnit_Extensions_Database_TestCase
{
    /**
     * This can be overridden when extending this class.
     * @var mixed Bootstrap resources required by the test.
     *            String for a single resource or array for multiple resources.
     */
    protected $bootstrapResources;

    /**
     * @var Zend_Db_Adapter_Abstract
     */
    protected $dbAdapter;

    /**
     * Path to current schemafile
     * @var string
     */
    protected $schemaFile;

    /**
     * @var PHPUnit_Extensions_Database_DB_DefaultDatabaseConnection
     */
    protected $conn = null;

    /**
     * @var Zend_Application_Bootstrap_Bootstrap
     */
    protected $bootstrap;

    /**
     * Create databse connection and load schema file
     *
     * @return PDO Database connection
     */
    final public function getConnection()
    {
        if (null === $this->conn) {
            $this->conn = $this->createDefaultDBConnection($this->dbAdapter->getConnection());
            Zend_Db_Table_Abstract::setDefaultAdapter($this->dbAdapter);
            $this->conn->getConnection()->exec(file_get_contents($this->schemaFile));
        }

        return $this->conn;
    }

    /**
     * Bootstrap application and required resources + create database adapter.
     */
    protected function setUp()
    {
        // Set configuration files
        $config = array(APPLICATION_PATH . '/configs/application.ini');
        if (file_exists(APPLICATION_PATH . '/configs/local.ini'))
            $config[] = APPLICATION_PATH . '/configs/local.ini';

        // Create application
        $application = new Zend_Application(
            APPLICATION_ENV,
            array('config' =>; $config)
        );

        // Bootstrap required resources.
        $application->bootstrap(array('config', 'multidb'));
        if (!empty($this->bootstrapResources)) {
            $application->bootstrap($this->bootstrapResources);
        }

        // Store bootstrap so we can later get resources from it.
        $this->bootstrap = $application->getBootstrap();

        $this->dbAdapter = $this->bootstrap->getPluginResource('multidb')->getDb('testdb');

        $options = $this->bootstrap->getOptions();
        $this->schemaFile = $options['resources']['multidb']['testdb']['testschema']['file'];

        parent::setUp();
    }
}
```

## Tests

In the actual test classes, fixtures can be used the same way as with *PHPUnit_Extensions_Database_TestCase*. You just need to implement a *getDataSet* method that loads the fixture. After that you can test the models or any other classes that rely on or modify data in the database and all of the data loaded by the fixtures and modified by the classes only live in the memory for as long the test is being executed. Here is a very simple example test, nothing special about it, just an example to complete the whole process:

```php?start_inline=1
class Application_Model_ExampleTest extends My_Test_PHPUnit_DatabaseTestCase
{
    protected function getDataSet()
    {
        return $this->createFlatXMLDataSet(dirname(__FILE__) . '/_fixtures/example-fixture.xml');
    }

    /**
     * @coversApplication_Model_Example::fetchAllExamples
     */
    public function testCanFetchExamples()
    {

        $model = new Application_Model_Example();
        $result = $model->fetchAllExamples();

        $this->assertCount(2, $result);
    }
}
```

## Database modifications

We use [MySQL Workbench](http://www.mysql.com/downloads/workbench/) to design and model the database and it's export function to generate MySQL create and alter scripts. Luckily though there is also a [plugin](http://www.henlich.de/software/sqlite-export-plugin-for-mysql-workbench/) that can be used to export the schema for SQLite. This way we only make database modifications in one place and then export multiple schema files. So far this has worked quite well and there have been no real problems with the modifications (and there have been quite a lot of them).

## Caveats

There are some caveats to consider when using different database engine for testing. Biggest one is that you can't use database specific functionality. MySQL has a lot of propriety features that are not available on other database engines and many developers that have been working mostly with MySQL might be accustomed to rely upon them. These features include such as *NOW()* function and *ON UPDATE CURRENT_TIMESTAMP* declaration. So you have to be careful about what MySQL functions you are using (especially date and time related functions), and maybe do a little bit more date handling on PHP side. The upside is that it forces you to write a code base that is more database engine agnostic and therefore more portable if there ever is a need to change the RDBMS for example.

## Conclusion

SQLite in-memory database is a great way to separate tests to a completely different database on a development machine where the main MySQL database is used for development. It requires no installation or configuration and should "just work" on any machine which makes running tests on different machines simple and easy.

This post focused on Zend Framework 1 implementation, but this concept should be quite easy to implement in any framework such as Symfony 2 or Zend Framework 2 and of course without any framework at all. Maybe I'll do a follow-up post about using this with another framework someday.

I am also planning a post about controller testing with fixtures that will describe how this SQLite database and fixtures can be used together with Zend Framework controller tests (Zend_Test_PHPUnit_ControllerTestCase) and its DOM based asserts.

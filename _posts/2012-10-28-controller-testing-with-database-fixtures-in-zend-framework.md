---
title: Controller testing with database fixtures in Zend Framework
categories:
  - Programming
tags:
  - PHP
  - PHPUnit
  - Testing
  - Zend Framework
---

Testing controllers with PHPUnit is not actually unit testing per definition, but more like automated integration testing, functional testing or acceptance testing. Whatever you want to call it, it definitely is useful at times because you can't always cover everything with unit tests. For example if you want to test control logic that is inside controller method or display logic that is in view templates, or just verify that all those different modules work together as expected. And to help achieve this, we have *[Zend_Test_PHPUnit_ControllerTestCase](http://framework.zend.com/manual/1.12/en/zend.test.phpunit.html)* which makes it easy to execute the whole MVC stack and assert against wide variety of things like redirects, HTTP response codes and presence &amp; contents of DOM elements in the produced HTML (and much more).

<!--more-->

But since most of the applications we develop are so heavily database driven, I faced the problem of wanting to test a controller that behaves differently depending on data stored in the database. So I started thinking that there must be a way to use fixtures for the database the same way as when testing models with [PHPUnit Database extension](http://www.phpunit.de/manual/3.6/en/database.html) (*PHPUnit_Extensions_Database_TestCase*). But since PHP does not have multiple inheritance, we can't extend both *Zend_Test_PHPUnit_ControllerTestCase* and *PHPUnit_Extensions_Database_TestCase*. So I started out to create a controller test case class that has support for fixtures the same way as the database test case. I mean, how hard can it be? Surely someone else has already done it and I don't have to reinvent the wheel!

After some time googling around and finding nothing very useful, I found myself diving deep in the internals of PHPUnit Database extension, working out what actually happen when you call *createDefaultDBConnection* method and what is actually done with the return value of the *getDataSet* method etc. The PHPUnit/Extensions/Database folder has 11 subfolders and 91 files inside it (PHPUnit version 3.6.10), so it is not the most simple extension and required wading through quite many classes, abstract classes and interfaces to finally arrive at a seemingly simple solution.

## Abstract class

```php?start_inline=1
/**
* PHPUnit test case extending Zend_Test_PHPUnit_ControllerTestCase. Sets up
* configuration for Zend_Application, so we do not need to repeat it for every test.
* Can also be used to load fixtures similar to PHPUnit_Extensions_Database_TestCase
*/
abstract class My_Test_PHPUnit_ControllerTestCase extends Zend_Test_PHPUnit_ControllerTestCase
{
    /**
     * @var bool Setup similar test database as DatabaseTestCase when true.
     */
    protected $_databaseTestCase = false;

    /**
     * Sets up bootstrap for the application.
     * This method is called before a test is executed.
     *
     * @return void
     */
    public function setUp()
    {
        // Set configuration files
        $config = array(APPLICATION_PATH . '/configs/application.ini');
        if (file_exists(APPLICATION_PATH . '/configs/local.ini'))
            $config[] = APPLICATION_PATH . '/configs/local.ini';

        // Create application
        $this->bootstrap = new Zend_Application(
            APPLICATION_ENV,
            array('config' => $config)
        );

        parent::setUp();

        if ($this->_databaseTestCase) {
          $this->_setupDatabase();
        }
    }

    /**
     * Setup test database and load fixture
     */
    protected function _setupDatabase()
    {
        $options = $this->bootstrap->getOptions();
        $schemaFile = $options['resources']['multidb']['testdb']['testschema']['file'];

        $db = $this->bootstrap->getBootstrap()->getPluginResource('multidb')->getDb('testdb');
        Zend_Db_Table_Abstract::setDefaultAdapter($db);
        $db->getConnection()->exec(file_get_contents($schemaFile));

        $connection = new PHPUnit_Extensions_Database_DB_DefaultDatabaseConnection($db->getConnection());

        $dataSet = $this->getDataSet();

        if ($dataSet instanceof PHPUnit_Extensions_Database_DataSet_IDataSet) {
          $setupOperation = PHPUnit_Extensions_Database_Operation_Factory::CLEAN_INSERT();
          $setupOperation->execute($connection, $dataSet);
        }
    }

    /**
     * Returns the test dataset.
     *
     * @return PHPUnit_Extensions_Database_DataSet_IDataSet
     */
    protected function getDataSet()
    {}
}
```

The **$_databaseTestCase** property is used to indicate if a test uses the database fixture functionality. Without setting it to true, the default database from the application configuration will be used when running the tests cases inheriting from this class.

The **setUp** method is called automatically by PHPUnit. In the **setUp** method we load our application configuration and bootstrap the application. After calling parents **setUp** method, we check if the running test is a database test case, and set up the database by calling **_setupDatabase** method.

The **_setupDatabase** method does all the heavy lifting by creating the database connection, registering it with PHPUnit Database extension by calling *PHPUnit_Extensions_Database_DB_DefaultDatabaseConnection* and loading the data set with a setup operation *PHPUnit_Extensions_Database_Operation_Factory::CLEAN_INSERT*.

There is some code specific to the way we are using Zend Framework *multidb* resource to set up an [SQLite database for testing](http://cvuorinen.net/2012/10/model-testing-using-sqlite-in-memory-database-with-zend-framework/) in the *_setupDatabase* method, but it should be easy enough to modify that part to work with any kind of database configuration.

## Test case


```php?start_inline=1
class Application_ExampleControllerTest extends My_Test_PHPUnit_ControllerTestCase
{
    protected $_databaseTestCase = true;
   
    protected function getDataSet()
    {
        return new PHPUnit_Extensions_Database_DataSet_FlatXmlDataSet(
            dirname(__FILE__) . '/_fixtures/example-items-fixture.xml'
        );
    }

    public function testListAction()
    {
        $params = array('action' => 'list', 'controller' => 'example');
        $urlParams = $this->urlizeOptions($params);
        $url = $this->url($urlParams);
        $this->dispatch($url);
       
        // Assert list table has 5 data rows
        $this->assertQueryCount('div.content table.list tbody tr', 5);
    }
   
    public function testUserCannotEditItemsThatAreNotTheirOwn()
    {
        // Create mock identity so we are authorized
        $identity = new stdClass();
        $identity->id = '1';
        $identity->username = 'user1';
        $identity->role = 'editor';
        Zend_Auth::getInstance()->getStorage()->write($identity);

        $params = array('action' => 'edit-item', 'controller' => 'example', 'id' => '1');
        $urlParams = $this->urlizeOptions($params);
        $url = $this->url($urlParams);
        $this->dispatch($url);
       
        // Assert Unauthorized response code
        $this->assertResponseCode(403);
       
        // Assert we do not have item edit form
        $this->assertQueryCount('form#item-edit', 0);
    }
   
    public function testUserCanEditOwnItems()
    {
        // Create mock identity so we are authorized
        $identity = new stdClass();
        $identity->id = '2';
        $identity->username = 'user2';
        $identity->role = 'editor';
        Zend_Auth::getInstance()->getStorage()->write($identity);
       
        $params = array('action' => 'edit-item', 'controller' => 'example', 'id' => '1');
        $urlParams = $this->urlizeOptions($params);
        $url = $this->url($urlParams);
        $this->dispatch($url);
       
        // Assert OK response code
        $this->assertResponseCode(200);
       
        // Assert we have item edit form
        $this->assertQueryCount('form#item-edit', 1);
    }
}
```

This is a simplified example test case class, that does a few basic tests to illustrate situations where fixtures support might be useful. All of these tests are dependent on the database and if we would be using the same database we are using for development to run these tests, the outcome of these tests are bound to change as the database changes.

## Conclusion

This post illustrates how the [PHPUnit Database extension](http://www.phpunit.de/manual/3.6/en/database.html) can be used without extending *PHPUnit_Extensions_Database_TestCase* by adding fixtures capabilities to any other PHPUnit test class. The method shown in this post might not be the most elegant solution or it might not work in every situation, but it has worked without issues in a current project I am working on for some time.

These code examples are for Zend Framework, but this same concept can just as easily be used with any other controller test class, for example [Symfony 2 *WebTestCase*](http://symfony.com/doc/2.0/book/testing.html#functional-tests).

## Bonus tip

It can sometimes be hard to write these test without actually seeing how the system reacts in every situation with the fixture database. If you want to take a look at how it really works and looks like in a browser, where you can click around and inspect the source etc., you can copy the database setup code to a controllers *init* method and give it a try.

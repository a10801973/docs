Testing
#######

CakePHP comes with comprehensive testing support built-in.  CakePHP comes with
integration for `PHPUnit <http://phpunit.de>`_.  In addition to the features
offered by PHPUnit, CakePHP offers some additional features to make testing
easier. This section will cover installing PHPUnit, and getting started with
Unit Testing, and how you can use the extensions that CakePHP offers.

Installing PHPUnit
==================

CakePHP uses PHPUnit as its underlying test framework.  PHPUnit is the de-facto
standard for unit testing in PHP.  It offers a deep and powerful set of features
for making sure your code does what you think it does.  PHPUnit can be installed
through the `pear installer <http://pear.php.net>`_.  To install PHPUnit run the
following::

    pear upgrade PEAR
    pear config-set auto_discover 1
    pear install pear.phpunit.de/PHPUnit

.. note::

    Depending on your system's configuration, you make need to run the previous
    commands with ``sudo``

Test database setup
===================

Remember to have a debug level of at least 1 in your ``app/Config/core.php`` 
file before running any tests.  Tests are not accessible via the web runner when
debug is equal to 0.  Before running any tests you should be sure to add a
``$test`` database configuration.  This configuration is used by CakePHP for
fixture tables and data::

    <?php
    public $test = array(
        'datasource' => 'Database/Mysql',
        'persistent' => false,
        'host' => 'dbhost',
        'login' => 'dblogin',
        'password' => 'dbpassword',
        'database' => 'test_database'
    );

.. note::

    Its a good idea to make the test database and your actual database
    different databases.  This will prevent any embarrassing mistakes later.

Checking the test setup
=======================

After installing PHPUnit and setting up your ``$test`` database configuration
you can make sure you're ready to write and run your own tests by running one of
the core tests. There are two built-in runners for testing, we'll start off by
using the web runner. The tests can then be accessed by browsing to
http://localhost/your_app/test.php. You should see a list of the core test
cases.  Click on the 'AllConfigure' test.  You should see a green bar with some
additional information about the tests run, and number passed.

Congratulations, you are now ready to start writing tests!

Test case conventions
=====================

Like most things in CakePHP, test cases have some conventions. concerning
tests:

#. PHP files containing tests should be in your
   ``app/Test/Case/[Type]`` directories.
#. The filenames of these files should end in ``Test.php`` instead
   of just .php.
#. The classes containing tests should extend ``CakeTestCase`` or
   ``PHPUnit_Framework_TestCase``.
#. Like other classnames, the test case classnames should match the filename.
   ``RouterTest.php`` should contain ``class RouterTest extends CakeTestCase``.
#. The name of any method containing a test (i.e. containing an
   assertion) should begin with ``test``, as in ``testPublished()``.
   You can also use the ``@test`` annotation to mark methods as test methods.

When you have created a test case, you can execute it by browsing
to ``http://localhost/you_app/test.php`` (depending on
how your specific setup looks). Click App test cases, and
then click the link to your specific file.  You can run tests from the command
line using the testsuite shell::

    ./Console/cake testsuite app Model/Post

For example, would run the tests for your Post model.

Creating your first test case
=============================

In the following example, we'll create a test case for a very simple helper
method.  The helper we're going to test will be formatting progress bar HTML.
Our helper looks like::

    <?php
    class ProgressHelper extends AppHelper {
        public function bar($value) {
            $width = round($value / 100) * 100;
            return sprintf(
                '<div class="progress-container">
                    <div class="progress-bar" style="width: %s%%"></div>
                </div>', $width);
        }
    }

This is a very simple example, but it will be useful to show how you can create
a simple test case.  After creating and saving our helper, we'll create the test
case file in ``app/Test/Case/View/Helper/ProgressHelperTest.php``.  In that file
we'll start with the following::

    <?php
    App::uses('ProgressHelper', 'View/Helper');
    App::uses('View', 'View');

    class ProgressHelperTest extends CakeTestCase {
        public function setUp() {
        
        }

        public function testBar() {

        }
    }

We'll flesh out this skeleton in a minute.  We've added two methods to start
with.  First is ``setUp()``.  This method is called before every *test* method
in a test case class.  Setup methods should initialize the objects needed for the
test, and do any configuration needed.  In our setup method we'll add the
following::

    <?php
    public function setUp() {
        parent::setUp();
        $View = new View();
        $this->Progress = new ProgressHelper($View);
    }

Calling the parent method is important in test cases, as CakeTestCase::setUp()
does a number things like backing up the values in :php:class:`Configure` and,
storing the paths in :php:class:`App`.

Next, we'll fill out the test method.  We'll use some assertions to ensure that
our code creates the output we expect::

    <?php
    public function testBar() {
        $result = $this->Progress->bar(90);
        $this->assertContains('width: 90%', $result);
        $this->assertContains('progress-bar', $result);

        $result = $this->Progress->bar(33.3333333);
        $this->assertContains('width: 33%', $result);
    }

The above test is a simple one but shows the potential benefit of using test
cases.  We use ``assertContains()`` to ensure that our helper is returning a
string that contains the content we expect.  If the result did not contain the
expected content the test would fail, and we would know that our code is
incorrect. 

By using test cases you can easily describe the relationship between a set of
known inputs and their expected output.  This helps you be more confident of the
code you're writing as you can easily check that the code you wrote fulfills the
expectations and assertions your tests make.  Additionally because tests are
code, they are easy to re-run whenever you make a change.  This helps prevent
the creation of new bugs.

.. _running-tests:

Running tests
=============

Once you have PHPUnit installed and some test cases written, you'll want to run
the test cases very frequently.  Its a good idea to run tests before committing
any changes to help ensure you haven't broken anything.

Running tests from a browser
----------------------------

CakePHP provides a web interface for running tests, so you can execute your
tests through a browser if you're more comfortable in that environment.  You can
access the web runner by going to ``http://localhost/your_app/test.php``.  The
exact location of test.php will change depending on your setup.  But the file is
at the same level as ``index.php``.

Once you've loaded up the test runner, you can navigate App, Core and Plugin test
suites.  Clicking an individual test case will run that test and display the
results.

Viewing code coverage
~~~~~~~~~~~~~~~~~~~~~

If you have `XDebug <http://xdebug.org>`_ installed, you can view code coverage
results.  Code coverage is useful for telling you what parts of your code your
tests do not reach. Coverage is useful for determining where you should add
tests in the future, and gives you one measurement to track your testing
progress with.

.. |Code Coverage| image:: /_static/img/code-coverage.png

|Code Coverage|

The inline code coverage uses green lines to indicate lines that have been run.
If you hover over a green line a tooltip will indicate which tests covered the
line. Lines in red did not run, and have not been exercised by your tests.  Grey
lines are considered unexecutable code by xdebug.

Filtering test cases
~~~~~~~~~~~~~~~~~~~~

When you have larger test cases, you will often want to run a subset of the test
methods when you are trying to work on a single failing case.  With the
webrunner you can use a GET parameter to filter test methods::

    /test.php?case=Console/ConsoleOutput&filter=Write

The filter parameter is used as a case-sensitve regular expression for filtering
which test methods to run.

.. _run-tests-from-command-line:

Running tests from command line
-------------------------------

CakePHP provides a ``testsuite`` shell for running tests.  You can run app, core
and plugin tests easily using the testsuite shell.  It accepts all the arguments
you would expect to find on the normal PHPUnit command line tool as well. From
your app directory you can do the following to run tests::

    # Run a model tests in the app
    ./Console/cake testsuite app Model/Article

    # Run a component test in a plugin
    ./Console/cake testsuite DebugKit Controller/Component/ToolbarComponent

    # Run the configure class test in CakePHP
    ./Console/cake testsuite core Core/Configure

.. note::

    If you are running tests that interact with the session its generally a good
    idea to use the ``--stderr`` option.  This will fix issues with tests
    failing because of headers_sent warnings.


Filtering test cases
~~~~~~~~~~~~~~~~~~~~

When you have larger test cases, you will often want to run a subset of the test
methods when you are trying to work on a single failing case.  With the
cli runner you can use an option to filter test methods::

    ./Console/cake testsuite core Core/ConsoleOutput --filter Write

The filter parameter is used as a case-sensitve regular expression for filtering
which test methods to run.

Generating code coverage
~~~~~~~~~~~~~~~~~~~~~~~~

You can generate code coverage reports from the command line using PHPUnit's
built-in code coverage tools.  PHPUnit will generate a set of static HTML files
containing the coverage results.  You can generate coverage for a test case by
doing the following::

    ./Console/cake testsuite app Model/Article --coverage-html webroot/coverage

This will put the coverage results in your application's webroot directory.  You
should be able to view the results by going to 
``http://localhost/your_app/coverage``.

Creating test suites
====================

If you want several of your tests to run at the same time, you can
creating a test suite. A testsuite is composed of several test cases.
``CakeTestSuite`` offers a few methods for easily creating test suites based on
the file system.  If we wanted to create a test suite for all our model tests we
could would create ``app/Test/Case/AllModelTest.php``. Put the following in it::

    <?php
    class AllModelTest extends CakeTestSuite {
        public static function suite() {
            $suite = new CakeTestSuite('All model tests');
            $suite->addTestDirectory(TESTS . 'Case' . DS . 'Model');
            return $suite;
        }
    }

The code above will group all test cases found in the
``/app/Test/Case/Model/`` folder. To add an individual file, use
``$suite->addTestFile($filename);``.  You can recursively add a directory
using::

    <?php
    $suite->addTestDirectoryRecursive(TESTS . 'Case' . DS . 'Controller');

Would recursively add all test cases in the ``app/Test/Case/Controller``
directory.

Fixtures
========

When testing code that depends on models and the database, one can use
**fixtures** as a way to generate temporary data tables loaded with sample data
that can be used by the test. The benefit of using fixtures is that your test
has no chance of disrupting live application data. In addition, you can begin
testing your code prior to actually developing live content for an application.

CakePHP uses the connection named ``$test`` in your ``app/Config/database.php``
configuration file. If this connection is not usable, an exception will be
raised and you will not be able to use database fixtures.

CakePHP performs the following during the course of a fixture based
test case:

#. Creates tables for each of the fixtures needed.
#. Populates tables with data, if data is provided in fixture.
#. Runs test methods.
#. Empties the fixture tables.
#. Removes fixture tables from database.

Creating fixtures
-----------------

When creating a fixture you will mainly define two things: how the
table is created (which fields are part of the table), and which
records will be initially populated to the table. Let's 
create our first fixture, that will be used to test our own Article
model. Create a file named ``ArticleFixture.php`` in your
``app/Test/Fixture`` directory, with the following content::

    <?php
    class ArticleFixture extends CakeTestFixture { 

          public $fields = array( 
              'id' => array('type' => 'integer', 'key' => 'primary'), 
              'title' => array('type' => 'string', 'length' => 255, 'null' => false), 
              'body' => 'text', 
              'published' => array('type' => 'integer', 'default' => '0', 'null' => false), 
              'created' => 'datetime', 
              'updated' => 'datetime' 
          ); 
          public $records = array( 
              array ('id' => 1, 'title' => 'First Article', 'body' => 'First Article Body', 'published' => '1', 'created' => '2007-03-18 10:39:23', 'updated' => '2007-03-18 10:41:31'), 
              array ('id' => 2, 'title' => 'Second Article', 'body' => 'Second Article Body', 'published' => '1', 'created' => '2007-03-18 10:41:23', 'updated' => '2007-03-18 10:43:31'), 
              array ('id' => 3, 'title' => 'Third Article', 'body' => 'Third Article Body', 'published' => '1', 'created' => '2007-03-18 10:43:23', 'updated' => '2007-03-18 10:45:31') 
          ); 
     } 

We use ``$fields`` to specify which fields will be part of this table,
and how they are defined. The format used to define these fields is
the same used with :php:class:`CakeSchema`.  The keys available for table
definition are:

type
    CakePHP internal data type. Currently supported: string (maps to
    VARCHAR), text (maps to TEXT), integer (maps to INT), float (maps
    to FLOAT), datetime (maps to DATETIME), timestamp (maps to
    TIMESTAMP), time (maps to TIME), date (maps to DATE), and binary
    (maps to BLOB)
key
    set to primary to make the field AUTO\_INCREMENT, and a PRIMARY KEY
    for the table.
length
    set to the specific length the field should take.
null
    set to either true (to allow NULLs) or false (to disallow NULLs)
default
    default value the field takes.

We can define a set of records that will be populated after the fixture table is
created. The format is fairly straight forward, ``$records`` is an array of
records.  Each item in ``$records`` should be a single row.  Inside each row,
should be an associative array of the columns and values for the row.  Just keep
in mind that each record in the $records array must have a key for **every**
field specified in the ``$fields`` array. If a field for a particular record needs
to have a NULL value, just specify the value of that key as NULL.

Importing table information and records
---------------------------------------

Your application may have already working models with real data
associated to them, and you might decide to test your application with
that data. It would be then a duplicate effort to have to define
the table definition and/or records on your fixtures. Fortunately,
there's a way for you to define that table definition and/or
records for a particular fixture come from an existing model or an
existing table.

Let's start with an example. Assuming you have a model named
Article available in your application (that maps to a table named
articles), change the example fixture given in the previous section
(``app/Test/Fixture/ArticleFixture.php``) to::

    <?php  
    class ArticleFixture extends CakeTestFixture { 
        public $import = 'Article'; 
    } 

This statement tells the test suite to import your table definition from the
table linked to the model called Article. You can use any model available in
your application. The statement will only import the Article schema, and  does
not import records. To import records you can do the following::

    <?php
    class ArticleFixture extends CakeTestFixture {
        public $import = array('model' => 'Article', 'records' => true);
    }

If on the other hand you have a table created but no model
available for it, you can specify that your import will take place
by reading that table information instead. For example::

    <?php  
    class ArticleFixture extends CakeTestFixture { 
        public $import = array('table' => 'articles'); 
    } 

Will import table definition from a table called 'articles' using
your CakePHP database connection named 'default'. If you want to
use a different connection use::

     <?php  
     class ArticleFixture extends CakeTestFixture { 
        public $import = array('table' => 'articles', 'connection' => 'other'); 
     } 

Since it uses your CakePHP database connection, if there's any
table prefix declared it will be automatically used when fetching
table information. The two snippets above do not import records
from the table. To force the fixture to also import its records,
change the import to::

     <?php  
     class ArticleFixture extends CakeTestFixture { 
        public $import = array('table' => 'articles', 'records' => true);
     } 

You can naturally import your table definition from an existing
model/table, but have your records defined directly on the fixture
as it was shown on previous section. For example::

     <?php
     class ArticleFixture extends CakeTestFixture {
          public $import = 'Article';
          public $records = array(
              array ('id' => 1, 'title' => 'First Article', 'body' => 'First Article Body', 'published' => '1', 'created' => '2007-03-18 10:39:23', 'updated' => '2007-03-18 10:41:31'),
              array ('id' => 2, 'title' => 'Second Article', 'body' => 'Second Article Body', 'published' => '1', 'created' => '2007-03-18 10:41:23', 'updated' => '2007-03-18 10:43:31'),
              array ('id' => 3, 'title' => 'Third Article', 'body' => 'Third Article Body', 'published' => '1', 'created' => '2007-03-18 10:43:23', 'updated' => '2007-03-18 10:45:31')
          );
       }

Loading fixtures in your test cases
-----------------------------------

After you've created your fixtures, you'll want to use them in your test cases.
In each test case you should load the fixtures you will need.  You should load a
fixture for every model that will have a query run against it.  To load fixtures
you define the ``$fixtures`` property in your model::

    <?php
    class ArticleTest extends CakeTestCase {
        public $fixtures = array('app.article', 'app.comment');
    }

The above will load the Article and Comment fixtures from the application's
Fixture directory.  You can also load fixtures from CakePHP core, or plugins::

    <?php
    class ArticleTest extends CakeTestCase {
        public $fixtures = array('plugin.debug_kit.article', 'core.comment');
    }

Using the ``core`` prefix will load fixtures from CakePHP, and using a plugin
name as the prefix, will load the fixture from the named plugin.

Testcase lifecycle callbacks
============================

Test cases have a number of lifecycle callbacks you can use when doing testing:

* ``setUp`` is called before every test method.  Should be used to create the
  objects that are going to be tested, and initialize any data for the test.
  Always remember to call ``parent::setUp()``
* ``tearDown`` is called after every test method.  Should be used to cleanup after
  the test is complete. Always remember to call ``parent::tearDown()``.
* ``setupBeforeClass`` is called once before test methods in a case are started.
  This method must be *static*.
* ``tearDownAfterClass`` is called once after test methods in a case are started.
  This method must be *static*.

Testing models
==============

Let's say we already have our Article model defined on
``app/Model/Article.php``, which looks like this::

     <?php  
     class Article extends AppModel { 
            public function published($fields = null) { 
                $params = array( 
                      'conditions' => array(
                            $this->name . '.published' => 1 
                      ),
                      'fields' => $fields
                ); 
                 
                return $this->find('all',$params); 
            }
     
     } 

We now want to set up a test that will use this model definition, but through
fixtures, to test some functionality in the model.  CakePHP test suite loads a
very minimum set of files (to keep tests isolated), so we have to start by
loading our model - in this case the Article model which we already defined.

Let's now create a file named ``ArticleTest.php`` in your
``app/Test/Case/Model`` directory, with the following contents::

     <?php  
      App::uses('Article', 'Model'); 
      
      class ArticleTestCase extends CakeTestCase { 
            public $fixtures = array('app.article');
      } 

In our test cases' variable ``$fixtures`` we define the set of fixtures that
we'll use.  You should remember to include all the fixtures that will have
queries run against them.

Creating a test method
----------------------

Let's now add a method to test the function published() in the
Article model. Edit the file
``app/Test/Case/Model/ArticleTest.php`` so it now looks like
this::

    <?php
    App::uses('Article', 'Model');

    class ArticleTestCase extends CakeTestCase {
        public $fixtures = array('app.article');

        public function setup() {
            parent::setUp();
            $this->Article = ClassRegistry::init('Article');
        }

        function testPublished() {
            $result = $this->Article->published(array('id', 'title'));
            $expected = array(
                array('Article' => array( 'id' => 1, 'title' => 'First Article' )),
                array('Article' => array( 'id' => 2, 'title' => 'Second Article' )),
                array('Article' => array( 'id' => 3, 'title' => 'Third Article' ))
            );

            $this->assertEquals($expected, $result);
        }
    }

You can see we have added a method called ``testPublished()``. We start by
creating an instance of our ``Article`` model, and then run our ``published()``
method. In ``$expected`` we set what we expect should be the proper result (that
we know since we have defined which records are initially populated to the
article table.) We test that the result equals our expectation by using the
``assertEquals`` method. See the :ref:`running-tests` section for more
information on how to run your test case.


Testing Helpers
===============

Since a decent amount of logic resides in Helper classes, it's
important to make sure those classes are covered by test cases.

Helper testing is a bit similar to the same approach for
Components. Suppose we have a helper called CurrencyRendererHelper
located in ``app/View/Helper/CurrencyRendererHelper.php`` with its
accompanying test case file located in
``app/Test/Case/Helper/CurrencyRendererTest.php``

Creating Helper test
--------------------

First of all we will define the responsibilities of our
``CurrencyRendererHelper``. Basically, it will have two methods just for
demonstration purpose:

function usd($amount)
    This function will receive the amount to render. It will take 2
    decimal digits filling empty space with zeros and prefix 'USD'.
function euro($amount)
    This function will do the same as usd() but prefix the output with
    'EUR'. Just to make it a bit more complex, we will also wrap the
    result in span tags::

        <span class="euro"></span> 

Let's create the tests first::

    <?php
    // Import the helper to be tested.
    App::uses('CurrencyRenderer', 'Helper');
    App::uses('View', 'View');
    
    class CurrencyRendererTest extends CakeTestCase {
        private $currencyRenderer = null;
    
        //Here we instantiate our helper, and all other helpers we need.
        public function setUp() {
            parent::setUp();
            $view = new View();
            $this->currencyRenderer = new CurrencyRendererHelper($view);
        }
    
        // testing usd() function.
        public function testUsd() {
            $this->assertEquals('USD 5.30', $this->currencyRenderer->usd(5.30));

            //We should always have 2 decimal digits.
            $this->assertEquals('USD 1.00', $this->currencyRenderer->usd(1));
            $this->assertEquals('USD 2.05', $this->currencyRenderer->usd(2.05));

            //Testing the thousands separator
            $this->assertEquals('USD 12,000.70', $this->currencyRenderer->usd(12000.70));
        }
    }

Here, we call ``usd()`` with different parameters and tell the test suite to
check if the returned values are equal to what is expected.

Executing the test now will result in errors (because currencyRendererHelper
doesn't even exist yet) showing that we have 3 fails.

Once we know what our method should do, we can write the method itself::

    <?php
    class CurrencyRendererHelper extends AppHelper {
        public function usd($amount) {
            return 'USD ' . number_format($amount, 2, '.', ',');
        }
    }

Here we set the decimal places to 2, decimal separator to dot, thousands
separator to comma, and prefix the formatted number with 'USD' string.

Save this in ``app/View/Helper/CurrencyRenderer.php`` and execute the test. You
should see a green bar and messaging indicating 4 passes.

Testing components
==================

Lets assume that we want to test a component called TransporterComponent, which
uses a model called Transporter to provide functionality for other controllers.
We will use four files:

-  A component called Transporters found in
   ``app/Controller/Component/TransporterComponent.php``
-  A model called Transporter found in
   ``app/Model/Transporter.php``
-  A fixture called TransporterTestFixture found in
   ``app/Test/Fixture/TransporterFixture.php``
-  The testing code found in
   ``app/Test/Case/TransporterTest.php``

Initializing the component
--------------------------

Since :doc:`/controllers/components`
we need a controller to access the data in the model.

If the ``startup()`` function of the component looks like this::

    <?php
    public function startup(Controller $controller){ 
        $this->Transporter = $controller->Transporter;
    }

then we can just design a really simple fake class::

    <?php
    class FakeTransporterController {}

and assign values into it like this::

    <?php
    $this->TransporterComponent = new TransporterComponent();
    $controller = new FakeTransporterController(); 
    $controller->Transporter = ClassRegistry::init('Transporter');
    $this->TransporterComponent->startup($controller);

Creating a test method
----------------------

Just create a class that extends CakeTestCase and start writing tests::

    <?php
    App::uses('TransporterComponent', 'Controller/Component');

    class TransporterTestCase extends CakeTestCase {
        public $fixtures = array('app.transporter');

        public function setUp() {
            parent::setUp();
            $this->TransporterComponent = new TransporterComponent();
            $controller = new FakeTransporterController();
            $controller->Transporter = ClassRegistry::init('Transporter');
            $this->TransporterComponentTest->startup($controller);
        }

        function testGetTransporter() {
            $result = $this->TransporterComponent->getTransporter("12345", "Sweden", "54321", "Sweden");
            $this->assertEquals(1, $result, "SP is best for 1xxxx-5xxxx");

            $result = $this->TransporterComponent->getTransporter("41234", "Sweden", "44321", "Sweden");
            $this->assertEquals(2, $result, "WSTS is best for 41xxx-44xxx");

            $result = $this->TransporterComponent->getTransporter("41001", "Sweden", "41870", "Sweden");
            $this->assertEquals(3, $result, "GL is best for 410xx-419xx");

            $result = $this->TransporterComponent->getTransporter("12345", "Sweden", "54321", "Norway");
            $this->assertEquals(0, $result, "No one can service Norway");
        }
    }

Testing controllers
===================

While you can test controller classes in a similar fashion to Helpers, Models,
and Components, CakePHP offers a specialized ``ControllerTestCase`` class.
Using this class as the base class for your controller test cases allows you to
use ``testAction()`` for simpler test cases.  ``ControllerTestCase`` allows you
to easily mock out components and models, as well as potentially difficult to
test methods like :php:meth:`~Controller::redirect()`.

Say you have a typical Articles controller, and its corresponding
model. The controller code looks like::

    <?php 
    class ArticlesController extends AppController { 
       public $helpers = array('Form', 'Html'); 
       
       function index($short = null) { 
         if (!empty($this->data)) { 
           $this->Article->save($this->data); 
         } 
         if (!empty($short)) { 
           $result = $this->Article->findAll(null, array('id', 'title'));
         } else { 
           $result = $this->Article->findAll(); 
         } 
     
         if (isset($this->params['requested'])) { 
           return $result; 
         } 
     
         $this->set('title', 'Articles'); 
         $this->set('articles', $result); 
       } 
    } 

Create a file named ``ArticlesControllerTest.php`` in your
``app/Test/Case/Controller`` directory and put the following inside::

    <?php
    class ArticlesControllerTest extends ControllerTestCase {

        public $fixtures = array('app.article');

        function testIndex() {
            $result = $this->testAction('/articles/index');
            debug($result);
        } 

        function testIndexShort() {
            $result = $this->testAction('/articles/index/short');
            debug($result);
        }

        function testIndexShortGetRenderedHtml() {
            $result = $this->testAction(
               '/articles/index/short',
                array('return' => 'render')
            );
            debug($result);
        }

        function testIndexShortGetViewVars() {
            $result = $this->testAction(
                '/articles/index/short',
                array('return' => 'vars')
            );
            debug($result);
        }

        function testIndexPostData() { 
            $data = array(
                'Article' => array(
                    'user_id' => 1,
                    'published' => 1,
                    'slug' => 'new-article',
                    'title' => 'New Article',
                    'body' => 'New Body'
                )
            );
            $result = $this->testAction(
                '/articles/index',
                array('data' => $data, 'method' => 'post')
            );
            debug($result);
        }
    }

This example shows a few of the ways you can use testAction to test your
controllers.  The first parameter of ``testAction`` should always be the URL you
want to test.  CakePHP will create a request and dispatch the controller and
action.

Simulating GET requests
------------------------

As seen in the ``testIndexPostData()`` example above, you can use
``testAction()`` to test POST actions as well as GET actions.  By supplying the
``data`` key, the request made to the controller will be POST.  By default all
requests will be POST requests.  You can simulate a GET request by setting the
method key::

    <?php
    function testAdding() {
        $data = array(
            'Post' => array(
                'title' => 'New post',
                'body' => 'Secret sauce'
            )
        );
        $this->testAction('/posts/add', array('data' => $data, 'method' => 'get'));
        // some assertions.
    }

The data key will bet used as query string parameters when simulating a GET
request.

Choosing the return type
------------------------

You can choose from a number of ways to inspect the success of your controller
action. Each offers a different way to ensure your code is doing what you
expect:

* ``vars`` Get the set view variables.
* ``view`` Get the rendered view, without a layout.
* ``contents`` Get the rendered view including the layout.
* ``result`` Get the return value of the controller action.  Useful 
  for testing requestAction methods.

The default value is ``result``.  As long as your return type is not ``result``
you can also access the various other return types as properties in the test
case::

    <?php
    public function testIndex() {
        $this->testAction('/posts/index');
        $this->assertIsA($this->vars['posts'], 'array');
    }


Using mocks with testAction
---------------------------

There will be times when you want to replace components or models with either
partially mocked objects or completely mocked objects.  You can do this by using
:php:meth:`~ControllerTestCase::generate()`. ``generate()`` takes the hard work
out of generating mocks on your controller. If you decide to generate a
controller to be used in testing, you can generate mocked versions of its models
and components along with it::

    <?php
    $Posts = $this->generate('Posts', array(
      'methods' => array(
        'isAuthorized'
      ),
      'models' => array(
        'Post' => array('save')
      ),
      'components' => array(
        'RequestHandler' => array('isPut'),
        'Email' => array('send'),
        'Session'
    )
    ));

The above would create a mocked ``PostsController``, stubbing out the ``isAuthorized``
method. The attached Post model will have ``save()`` stubbed, and the attached
components would have their respective methods stubbed. You can choose to stub
an entire class by not passing methods to it, like Session in the example above.

Generated controllers are automatically used as the testing controller to test.
To enable automatic generation, set the ``autoMock`` variable on the test case to
true. If ``autoMock`` is false, your original controller will be used in the test.

The response object in the generated controller is always replaced with a mock
that does not send headers. After using ``generate()`` or ``testAction()`` you
can access the controller object at ``$this->controller``.

A more complex example
----------------------

In its simplest form, testAction will run PostsController::index() on your
testing controller (or an automatically generated one), including all of the
mocked models and components. The results of the test are stored in the vars,
contents, view, and return properties. Also available is a headers property which
gives you access to the headers that would have been sent, allowing you to check
for redirects::

    <?php
    function testAdd() {
        $Posts = $this->generate('Posts', array(
            'components' => array(
              'Session',
              'Email' => array('send')
            )
        ));
        $Posts->Session->expects($this->once())->method('setFlash');
        $Posts->Email->expects($this->once())->method('send')
            ->will($this->returnValue(true));

        $this->testAction('/posts/add', array(
            'data' => array(
              'Post' => array('name' => 'New Post')
            )
        ));

        $this->assertEquals($this->headers['Location'], '/posts/index');
        $this->assertEquals($this->vars['post']['Post']['name'], 'New Post');
        $this->assertRegExp('/<html/', $this->contents);
        $this->assertRegExp('/<form/', $this->view);
    }

This example shows a slightly more complex use of the new testAction and
generate() methods.  First, we generate a testing controller and mock the
:php:class:`SessionComponent`. Now that the SessionComponent is mocked, we have the ability
to run testing methods on it. Assuming PostsController::add() redirects us to
index, sends an email and sets a flash message, the test will pass. For the sake
of example, we also check to see if the layout was loaded by checking the entire
rendered contents, and checks the view for a form tag. As you can see, your
freedom to test controllers and easily mock its classes is greatly expanded with
these changes.


Creating tests for plugins
==========================

Tests for plugins are created in their own directory inside the
plugins folder.::

    /app
         /Plugin
             /Blog
                 /Test
                    /Case
                    /Fixture

They work just like normal tests but you have to remember to use
the naming conventions for plugins when importing classes. This is
an example of a testcase for the ``BlogPost`` model from the plugins
chapter of this manual. A difference from other tests is in the
first line where 'Blog.BlogPost' is imported. You also need to
prefix your plugin fixtures with ``plugin.blog.blog_post``::

    <?php 
    App::uses('Blog.BlogPost', 'Model');

    class BlogPostTest extends CakeTestCase {

        // Plugin fixtures located in /app/Plugin/Blog/Test/Fixture/
        public $fixtures = array('plugin.pizza.pizza_order');
        public $PizzaOrderTest;

        function testSomething() {
            // ClassRegistry makes the model use the test database connection
            $this->BlogPost = ClassRegistry::init('Blog.BlogPost');

            // do some useful test here
            $this->assertTrue(is_object($this->BlogPost));
        }
    }

If you want to use plugin fixtures in the app tests you can
reference them using ``plugin.pluginName.fixtureName`` syntax in the
``$fixtures`` array.

Integration with Jenkins
========================

`Jenkins <http://jenkins-ci.org>`_ is a continuous integration server, that can
help you automate the running of your test cases.  This helps ensure that all
your tests stay passing and your application is always ready.

Integrating a CakePHP application with Jenkins is fairly straightforward.  The
following assumes you've already installed Jenkins on \*nix system, and are able
to administer it.  You also know how to create jobs, and run builds.  If you are
unsure of any of these, refer to the `Jenkins documentation <http://jenkins-ci.org/>`_ .

Create a job
------------

Start off by creating a job for your application, and connect your repository
so that jenkins can access your code.

Add test database config
------------------------

Using a separate database just for Jenkins is generally a good idea, as it stops
bleedthrough and avoids a number of basic problems.  Once you've created a new
database in a database server that jenkins can access (usually localhost).  Add
a *shell script step* to the build that contains the following::

    cat > app/Config/database.php <<'DATABASE_PHP'
    <?php
    class DATABASE_CONFIG {
      public $test = array(
        'datasource' => 'Database/Mysql',
        'host' => 'localhost',
        'database' => 'jenkins_test',
        'login' => 'jenkins',
        'password' => 'cakephp_jenkins',
        'encoding' => 'utf8'
      );
    }
    DATABASE_PHP
 
This ensures that you'll always have the correct database configuration that
Jenkins requires.  Do the same for any other configuration files you need to.
Its often a good idea to drop and re-create the database before each build as
well.  This insulates you from chained failures, where one broken build causes
others to fail.  Add another *shell script step* to the build that contains the
following::

    mysql -u jenkins -pcakephp_jenkins -e 'DROP DATABASE jenkins_test; CREATE DATABASE jenkins_test';

Add your tests
--------------

Add another *shell script step* to your build.  In this step run the tests for
your application.  Creating a junit log file, or clover coverage is often a nice
bonus, as it gives you a nice graphical view of your testing results::

    app/Console/cake testsuite app AllTests \
    --stderr \
    --log-junit junit.xml
    --clover-coverage clover.xml

If you use clover coverage, or the junit results, make sure to configure those
in Jenkins as well. Failing to configure those steps will mean you won't see the results.

Run a build
-----------

You should be able to run a build now.  Check the console output and make any
necessary changes to get a passing build.



.. meta::
    :title lang=en: Testing
    :keywords lang=en: web runner,phpunit,test database,database configuration,database setup,database test,public test,test framework,running one,test setup,de facto standard,pear,runners,array,databases,cakephp,php,integration
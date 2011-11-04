# SpiffyDoctrine Module for Zend Framework 2
* THIS BRANCH REQUIRES RALPH SCHINDLER'S HOTFIX FOR ZF2/DI, https://github.com/ralphschindler/zf2.git (a8de0890e216e9826be3414a818c333e2f307028) *

The SpiffyDoctrine module intends to integrate Doctrine 2 ORM with Zend Framework 2 quickly and easily. The following features are intended to work out of the box: 
  
  - Multiple ORM entity managers
  - Multiple DBAL connections
  - Caches for metadata, queries and resultsets
  - Using a SQL logger
  - Custom dql functions, additional hydration modes
  - Named DQL and native queries
  - Multiple metadata drivers
  - Annotations registries initialization (such as Gedmo DoctrineExtensions).
  - Validators for EntityExists and NoEntityExists.
  - Authentication adapter for Zend\Authenticator.
  - Support for using existing PDO connections.
  
## Requirements
  - Zend Framework 2

## Installation
The simplest way to install is to clone the repository into your /modules directory add the 
SpiffyDoctrine key to your modules array before your Application module key.

  1. cd my/project/folder
  2. git clone https://SpiffyJr@github.com/SpiffyJr/SpiffyDoctrine.git modules/SpiffyDoctrine --recursive
  3. open my/project/folder/configs/application.config.php and add 'SpiffyDoctrine' to your 'modules' parameter.
  4. Alter the configuration (most likely your connection and entities path(s)) by adding the required changes to 
     my/project/folder/modules/Application/module.config.php.

## Example standard configuration
    // modules/Application/module.config.php
    'di' => array(
        'instance' => array(
            'doctrine' => array(
                'parameters' => array(
                    'conn' => array(
                        'driver'   => 'pdo_mysql',
                        'host'     => 'localhost',
                        'port'     => '3306', 
                        'user'     => 'USERNAME',
                        'password' => 'PASSWORD',
                        'dbname'   => 'DBNAME',
                    ),
                    'config' => array(
                        'metadata-driver-impl' => array(
                            'doctrine-annotationdriver' => array(
                                'namespace' => 'My\Entity\Namespace',
                                'paths'     => array('path/to/your/entities')
                            )
                        )
                    )
                ),
            ),
        )
    )


## Available locator items
Following locator items are preconfigured with this module:

  - doctrine', a SpiffyDoctrine\Servier\Doctrine instance

## Usage
Access the entity manager using the following locator: 

    $em = $this->getLocator()->get('doctrine')->getEntityManager();

## Doctrine CLI
The Doctrine CLI has been pre-configured and is available in SpiffyDoctrine\bin. It should work as
is without any special configuration required.

## Tuning for production
Tuning the system for production should be as simple as setting the following in your
configuration (example presumes you have APC installed).

    'di' => array(
        'instance' => array(
            'doctrine' => array(
                'parameters' => array(
                    'config' => array(
                        'auto-generate-proxies' => true,
                        'metadata-driver-impl' => array(
                            'doctrine-annotationdriver' => array(
                                'cache-class' => 'Doctrine\Common\Cache\ApcCache'
                            )
                        ),
                        'metadata-cache-impl' => 'Doctrine\Common\Cache\ApcCache',
                        'query-cache-impl'    => 'Doctrine\Common\Cache\ApcCache',
                        'result-cache-impl'   => 'Doctrine\Common\Cache\ApcCache'
                    ),
                ),
            ),
        ),

## Extra goodies
The items listed below are entirely optional and are intended to enhance ZF2/D2 integration.

### Using a PDO connection
Using a PDO connection requires some modifications to your module configuration. In your
application module config add the following.

    'di' => array( 
        'instance' => array(
            'doctrine-connection' => array(
                'parameters' => array(
                    'config' => 'doctrine-configuration',
                    'eventManager' => 'doctrine-eventmanager',
                    'pdo' => 'doctrine-pdo'
                ),
            ),
            'doctrine-pdo' => array(
                'parameters' => array(
                    'dsn' => 'mysql:dbname=SOME_DB;host=SOME_HOST',
                    'username' => 'SOME_USER',
                    'passwd' => 'SOME_PASSWORD',
                    'driver_options' => array(
                        \PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES \'UTF8\''
                    ),
                )
            ),
        )
    )

### EntityExists and NoEntityExists Validators
The EntityExists and NoEntityExists are validators similar to Zend\Validator\Db validators. You can 
pass a variety of options to determine validity. The most basic use case requires an entity manager (em),
an entity, and a field. You also have the option of specifying a query_builder Closure to use if you
want to fine tune the results. 

    $validator = new \SpiffyDoctrine\Validator\NoEntityExists(array(
       'em' => $this->getLocator()->get('doctrine')->getEntityManager(),
       'entity' => 'SpiffyUser\Entity\User',
       'field' => 'username',
       'query_builder' => function($er) {
           return $er->createQueryBuilder('q');
       }
    ));
    var_dump($validator->isValid('test'));        
        
### Authentication adapter for Zend\Authentication
The authentication adapter is intended to provide an adapter for Zend\Authenticator. It works much
like the DbTable adapter in the core framework. You must provide the entity manager instance,
entity name, identity field, and credential field.

    $adapter = new \SpiffyDoctrine\Authentication\Adapter\DoctrineEntity(
        $this->getLocator()->get('doctrine')->getEntityManager(), // entity manager
        'Application\Test\Entity',
        'username', // optional, default shown
        'password'  // optional, default shown
    );
    $adapter->setIdentity('username');
    $adapter->setCredential('password');
    $result = $adapter->authenticate();
    
    var_dump($result);
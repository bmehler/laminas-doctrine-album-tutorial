# laminas-mvc-tutorial

Dies ist ein Laminas MVC Tutorial. Derzeit arbeite ich mich in das Laminas PHP Framework ein und möchte meine Erkenntnisse hier in einem Tutorial zusammenfassen.

Das Tutorial richtet sich nach der [Laminas Album Applikation](https://docs.laminas.dev/tutorials/getting-started/overview/).

Bei diesem Tutorial möchte ich jedoch Doctrine ORM als Object Relationship Mapper für die Kommunikation zwischen der Entity und der Datenbank verwenden.

Die Dokumentation zu Doctrine für Laminas findet ihr [hier](https://www.doctrine-project.org/projects/doctrine-orm-module/en/5.3/index.html)

Als nützliches Tutorial habe ich dieses gefunden: [Tutorial](https://samsonasik.wordpress.com/2013/04/10/zend-framework-2-generate-doctrine-entities-from-existing-database-using-doctrinemodule-and-doctrineormmodule/)

## Installation

Mit dem Composer installiert man Laminas wie folgt:
```bash
 composer create-project -s dev laminas/laminas-mvc-skeleton laminas-doctrine
```

Doctrine ORM könnt ihr euch mit folgendem Composer Befehl holen:
```bash
cd laminas-doctrine
composer require doctrine/doctrine-orm-module
composer require doctrine/doctrine-module
composer require doctrine/annotations
composer require doctrine/annotations
composer require laminas/laminas-i18n
composer require doctrine/persistence
```
Nun steht die Konfiguration in Laminas an. Dies geht wie folgt:
```php
config/global.php

return [
    'doctrine' => [
        'connection' => [
            // default connection name
            'orm_default' => [
                'driverClass' => \Doctrine\DBAL\Driver\PDO\MySQL\Driver::class,
                'params' => [
                    'host'     => 'localhost',
                    'port'     => '3306',
                    'user'     => 'username',
                    'password' => 'password',
                    'dbname'   => 'database',
                ],
            ],
        ],
    ],
];
```
Als nächstes erstellt ihr eine Datenbank in Mysql. Als Kollation könnt ihr utf8_unicode_ci verwenden.
Bei mir heisst die Datenbank:
```bash
laminas-doctrine
```
Ihr seit hier aber offen.

## Die Album Applikation

Da ich hier mit Doctrine arbeite ändert sich der Aufbau ein wenig (siehe Entity)

### Aufbau der Applikation
```bash
   /module
        /Album/
            /config
                 module.config.php
            /src
                /Enitiy/Album.php
                /Controller
                /Form
                Module.php
            /view
```

Die weiteren Schritte könnt ihr der [Laminas Dokumentation](https://docs.laminas.dev/tutorials/getting-started/modules/) entnehmen.

## Die ORM Configuration im Module
Wir legen im Ordner config s.o. die Datei module.config.php.
Diese Datei sollte folgendes beinhalten und führt die Configuration der Entity Album weiter (siehe Punkt doctrine):

```php
<?php

<?php
namespace Album;

use Laminas\Router\Http\Segment;
use Doctrine\ORM\Mapping\Driver\AnnotationDriver;
use Doctrine\ORM\Mapping\Driver\DriverChain;
use Laminas\ServiceManager\Factory\InvokableFactory;
use Laminas\ServiceManager\AbstractFactory\ReflectionBasedAbstractFactory;

return [
    'doctrine' => [
        'driver' => [
            // defines an annotation driver with two paths, and names it `my_annotation_driver`
            'Album_driver' => [
                'class' => AnnotationDriver::class,
                'cache' => 'array',
                'paths' => [
                    __DIR__ . '/../src/Entity'
                ],
            ],

            // default metadata driver, aggregates all other drivers into a single one.
            // Override `orm_default` only if you know what you're doing
            'orm_default' => [
                'class'   => DriverChain::class,
                'drivers' => [
                    // register `my_annotation_driver` for any entity under namespace `My\Namespace`
                    'Album\Entity' =>  'Album_driver'
                ],
            ],
        ],
    ],
    'controllers' => [
        'factories' => [
            Controller\AlbumController::class => ReflectionBasedAbstractFactory::class,
        ],
    ],
     // The following section is new and should be added to your file:
    'router' => [
        'routes' => [
            'album' => [
                'type'    => Segment::class,
                'options' => [
                    'route' => '/album[/:action[/:id]]',
                    'constraints' => [
                        'action' => '[a-zA-Z][a-zA-Z0-9_-]*',
                        'id'     => '[0-9]+',
                    ],
                    'defaults' => [
                        'controller' => Controller\AlbumController::class,
                        'action'     => 'index',
                    ],
                ],
            ],
        ],
    ],
    'view_manager' => [
        'template_path_stack' => [
            'album' => __DIR__ . '/../view',
        ],
    ],
];
```

## Die Album Entity
Wir legen eine Entity Album an. Siehe oben liegt die Datei in:
```php
module/Album/src/Entity/Album.php

<?php

namespace Album\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * Album 
 * 
 * @ORM\Entity
 * @ORM\Table(name="album")
 */
class Album
{
    /**
     * @ORM\Id
     * @ORM\Column(type="integer")
     * @ORM\GeneratedValue
     */
    public $id;
    /**
     * @ORM\Column(name="artist", type="string", length=50, nullable=true)
     */
    public $artist;
     /**
     * @ORM\Column(name="title", type="string", length=50, nullable=true)
     */
    public $title;

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getTitle(): ?string
    {
        return $this->title;
    }

    public function setTitle(string $title): void
    {
        $this->title = $title;
    }

    public function getArtist(): ?string
    {
        return $this->artist;
    }

    public function setArtist(string $artist): void
    {
        $this->artist = $artist;
    }
}
```

## Doctrine
Die Konfiguration und die Erstellung einer Entity sind somit abgeschlossen. Nur die Verbindung zwischen Entity und Datenbank hat noch nicht stattgefunden.
Dies übernimmt die Doctrine Kommandozeile. Hierzu öffnet ihr das Terminal (z.b. Visual Studio Code - Anzeigen Terminal) 

Ihr erhaltet eine Liste mit den möglichen Befehlen zu Doctrine wie folgt:
```bash
 php vendor/bin/doctrine-module list
```
Nun könnt ihr überprüfen, ob das Mapping und das Datenbank Schema ordnungsgemäß gesynct wurden. Hier sollte ein Fehler erscheinen.
```bash
php vendor/bin/doctrine-module orm:validate-schema
```
Nachdem ihr
```bash
php vendor/bin/doctrine-module orm:schema-tool:create
```
sollte der Fehler nicht mehr auftauchen.

Mit diesem Befehl könnt ihr euch zuerst das SQL ausgeben lassen:
```bash
php vendor/bin/doctrine-module orm:schema-tool:create --dump-sql

CREATE TABLE album (id INT AUTO_INCREMENT NOT NULL, artist VARCHAR(50) DEFAULT NULL, title VARCHAR(50) DEFAULT NULL, PRIMARY KEY(id)) DEFAULT CHARACTER SET utf8 COLLATE `utf8_unicode_ci` ENGINE = InnoDB;
```
könnt ihr euch den SQL Code anzeigen lassen, welcher die Tabelle album in der Datenbank anlegt. Dies dient lediglich der Vorschau.

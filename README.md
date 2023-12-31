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

Hierbei ist der Pfad  ist. Also src und dann Entity. Sonst bekommt man den Fehler "Class does not exit".

```bash
module/Album/src/Entity/Album.php
```

Die weiteren Schritte könnt ihr der [Laminas Dokumentation](https://docs.laminas.dev/tutorials/getting-started/modules/) entnehmen.

## Die ORM Configuration im Module
```bash
/module/Album/config/module.config.php
```
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
```bash
/module/Album/src/Entity/Album.php
```
```php
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

## Controllers

Nun erstellt ihr den Controller. Dieser sieht wie folgt aus:
```bash
/module/Album/src/Controller/AlbumController.php
```
```php
<?php
namespace Album\Controller;

use Album\Entity\Album;
use Album\Form\CreateAlbumForm; -> siehe nächster Abschnitt
use Album\Form\EditAlbumForm; -> sieht nächster Abschnitt
use Doctrine\ORM\Annotations;
use Doctrine\ORM\EntityManager;
use Laminas\Mvc\Controller\AbstractActionController;
use Laminas\View\Model\ViewModel;

class AlbumController extends AbstractActionController
{
    private EntityManager $entityManager;
    
    public function __construct(EntityManager $entityManager)
    {
        $this->entityManager = $entityManager;
    }
    
    public function indexAction()
    {
        $findAll = $this->entityManager->getRepository(Album::class)->findAll();
        
        return new ViewModel([
            'albums' => $findAll,
        ]);
    }

    public function addAction()
    {
        // Create the form and inject the EntityManager
    $form = new CreateAlbumForm($this->entityManager);

    // Create a new, empty entity and bind it to the form
    $album = new Album();
    $form->bind($album);

    if ($this->request->isPost()) {
        $form->setData($this->request->getPost());

        if ($form->isValid()) {
            $this->entityManager->persist($album);
            $this->entityManager->flush();
            return $this->redirect()->toRoute('album');
        }
    }

    return ['form' => $form];
    }

    public function editAction()
    {
        $form = new EditAlbumForm($this->entityManager);
        $id = (int) $this->params()->fromRoute('id' , '0');
        
        $album = $this->entityManager->getRepository(Album::class)->find($id);
        
        $form->bind($album);

        if ($this->request->isPost()) {
            $form->setData($this->request->getPost());

            if ($form->isValid()) {
                $this->entityManager->flush();
                return $this->redirect()->toRoute('album');
            }
        }

        return ['id' => $id, 'form' => $form];
    }

    public function deleteAction()
    {
        $id = (int) $this->params()->fromRoute('id', 0);
        $album = $this->entityManager->getRepository(Album::class)->find($id);
        
        if (!$id) {
            return $this->redirect()->toRoute('album');
        }

        $request = $this->getRequest();
        if ($request->isPost()) {
            $del = $request->getPost('del', 'No');

            if ($del == 'Yes') {
                $id = (int) $request->getPost('id');
                $this->entityManager->remove($album);
                $this->entityManager->flush();
            }

            // Redirect to list of albums
            return $this->redirect()->toRoute('album');
        }

        return [
            'id'    => $id,
            'album' => $album,
        ];
    }
}
```

## Forms
In einem nächsten Schritt erstellt ihr die Forms. Diese sehen wir folgt aus:

### Album Form Fieldset
```bash
/module/Album/src/Form/AlbumFormFieldset.php
```
```php
<?php
namespace Album\Form;

use Album\Entity\Album;
use Doctrine\Laminas\Hydrator\DoctrineObject as DoctrineHydrator;
use Doctrine\Persistence\ObjectManager;
use Laminas\Form\Element\Text;
use Laminas\Form\Fieldset;
use Laminas\InputFilter\InputFilterProviderInterface;
use Laminas\Validator;
use Laminas\Filter;

class AlbumFormFieldset extends Fieldset implements InputFilterProviderInterface
{
    public function __construct(ObjectManager $objectManager)
    {
        parent::__construct('album');

        $this->setHydrator(new DoctrineHydrator($objectManager))
             ->setObject(new Album());

         $this->add([
            'name' => 'id',
            'type' => 'hidden',
        ]);

        $this->add([
            'name' => 'title',
            'type' => Text::class,
            'options' => [
                'label' => 'Title'
            ]
        ]);

        $this->add([
            'name' => 'artist',
            'type' => Text::class,
            'options' => [
                'label' => 'Artist'
            ]
        ]);

        $this->add([
            'name' => 'submit',
            'type' => 'submit',
            'attributes' => [
                'value' => 'Add',
                'id'    => 'submitbutton',
            ],
        ]);
    }

    public function getInputFilterSpecification()
    {
        return [
            'title' => [
                'required' => true,
                'filters'  => [
                    ['name' => Filter\StringTrim::class],
                ],
                'validators' => [
                    [
                        'name' => Validator\StringLength::class,
                        'options' => [
                            'min' => 2,
                            'max' => 50
                        ],
                    ],
                ],
            ],
            'artist' => [
                'required' => true,
                'filters'  => [
                    ['name' => Filter\StringTrim::class],
                ],
                'validators' => [
                    [
                        'name' => Validator\StringLength::class,
                        'options' => [
                            'min' => 2,
                            'max' => 50,
                        ],
                    ],
                ],
            ],
        ];
    }
}
```
### Album Create Album Form
```bash
/module/Album/src/Form/CreateAlbumForm.php
```
```php
/modules/Album/src/Form/CreateAlbumForm.php
<?php
namespace Album\Form;

use Doctrine\Laminas\Hydrator\DoctrineObject as DoctrineHydrator;
use Doctrine\Persistence\ObjectManager;
use Laminas\Form\Form;

class CreateAlbumForm extends Form
{
    public function __construct(ObjectManager $objectManager)
    {
        parent::__construct('create-album-form');

        // The form will hydrate an object of type "BlogPost"
        $this->setHydrator(new DoctrineHydrator($objectManager));

        // Add the BlogPost fieldset, and set it as the base fieldset
        $albumFieldset = new AlbumFormFieldset($objectManager);
        $albumFieldset->setUseAsBaseFieldset(true);
        $this->add($albumFieldset);
    }
}
```

### Album Edit Album Form
```bash
/module/Album/src/Form/EditAlbumForm.php
```
```php
<?php
namespace Album\Form;

use Doctrine\Laminas\Hydrator\DoctrineObject as DoctrineHydrator;
use Doctrine\Persistence\ObjectManager;
use Laminas\Form\Form;

class EditAlbumForm extends Form
{
    public function __construct(ObjectManager $objectManager)
    {
        parent::__construct('edit-album-form');

        // The form will hydrate an object of type "BlogPost"
        $this->setHydrator(new DoctrineHydrator($objectManager));

        // Add the BlogPost fieldset, and set it as the base fieldset
        $albumFieldset = new AlbumFormFieldset($objectManager);
        $albumFieldset->setUseAsBaseFieldset(true);
        $this->add($albumFieldset);
    }
}
```

## Views

### Index View

```bash
module/Album/src/view/album/album/index.phtml
```
```php
<?php
$title = 'My albums';
$this->headTitle($title);
?>
<h1><?= $this->escapeHtml($title) ?></h1>
<p>
    <a href="<?= $this->url('album', ['action' => 'add']) ?>">Add new album</a>
</p>

<table class="table">
<tr>
    <th>Title</th>
    <th>Artist</th>
    <th>&nbsp;</th>
</tr>
<?php foreach($albums as $album): ?>
 <tr>
        <td><?= $this->escapeHtml($album->title) ?></td>
        <td><?= $this->escapeHtml($album->artist) ?></td>
        <td>
            <a href="<?= $this->url('album', ['action' => 'edit', 'id' => $album->id]) ?>">Edit</a>
            <a href="<?= $this->url('album', ['action' => 'delete', 'id' => $album->id]) ?>">Delete</a>
        </td>
    </tr>
<?php endforeach; ?>
</table>
```
### Add View
```bash
module/Album/src/view/album/album/add.phtml
```
#### Hinweis
Die Feldwerte bekommnt man mit (Modul und Entity):
```bash
$form->get('album')->get("title");
```
```php
<?php
$title = 'Add new album';
$this->headTitle($title);
?>
<h1><?= $this->escapeHtml($title) ?></h1>
<?php

$form = $this->form;

$title = $form->get('album')->get("title");
$title->setAttribute('class', 'form-control');
$title->setAttribute('placeholder', 'Album title');

$artist = $form->get('album')->get('artist');
$artist->setAttribute('class', 'form-control');
$artist->setAttribute('placeholder', 'Artist');

$submit =$form->get('album')->get('submit');
$submit->setAttribute('class', 'btn btn-primary');

$form->setAttribute('action', $this->url('album', ['action' => 'add']));
$form->prepare();

echo $this->form()->openTag($form); ?>

<div class="form-group">
    <?= $this->formLabel($title) ?>
    <?= $this->formElement($title) ?>
    <?= $this->formElementErrors($title); ?>
</div>

<div class="form-group">
    <?= $this->formLabel($artist) ?>
    <?= $this->formElement($artist) ?>
    <?= $this->formElementErrors($artist); ?>
</div>

<?php
echo $this->formSubmit($submit);
echo $this->formHidden($form->get('album')->get("id"));
echo $this->form()->closeTag();
?>
```

### Edit View
```bash
module/Album/src/view/album/album/edit.phtml
```
```php
<?php
$title = 'Edit album';
$this->headTitle($title);
?>
<h1><?= $this->escapeHtml($title) ?></h1>
<?php

$form = $this->form;

$title = $form->get('album')->get("title");
$title->setAttribute('class', 'form-control');
$title->setAttribute('placeholder', 'Album title');

$artist = $form->get('album')->get('artist');
$artist->setAttribute('class', 'form-control');
$artist->setAttribute('placeholder', 'Artist');

$submit =$form->get('album')->get('submit');
$submit->setAttribute('class', 'btn btn-primary');

$form->setAttribute('action', $this->url('album', [
    'action' => 'edit',
    'id'     => $id,
]));

$form->prepare();

echo $this->form()->openTag($form); ?>

<div class="form-group">
    <?= $this->formLabel($title) ?>
    <?= $this->formElement($title) ?>
    <?= $this->formElementErrors($title); ?>
</div>

<div class="form-group">
    <?= $this->formLabel($artist) ?>
    <?= $this->formElement($artist) ?>
    <?= $this->formElementErrors($title); ?>
</div>

<?php
echo $this->formSubmit($submit);
echo $this->formHidden($form->get('album')->get("id"));
echo $this->form()->closeTag();
?>
```
### Delete View
```bash
module/Album/src/view/album/album/delete.phtml
```
```php
<?php
$title = 'Delete album';
$url   = $this->url('album', ['action' => 'delete', 'id' => $id]);

$this->headTitle($title);
?>
<h1><?= $this->escapeHtml($title) ?></h1>

<p>
    Are you sure that you want to delete
    "<?= $this->escapeHtml($album->title) ?>" by
    "<?= $this->escapeHtml($album->artist) ?>"?
</p>

<form action="<?= $url ?>" method="post">
<div class="form-group">
    <input type="hidden" name="id" value="<?= (int) $album->id ?>" />
    <input type="submit" class="btn btn-danger" name="del" value="Yes" />
    <input type="submit" class="btn btn-success" name="del" value="No" />
</div>
</form>
```

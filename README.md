# The Illuminate Queue package for CodeIgniter 4

Inspired from https://github.com/agungsugiarto/codeigniter4-eloquent

## Instalation

Include this package via Composer:

```console
composer require masrodjie/codeigniter4-queue
```

## Setup services queue

Add redis config in .env
```
REDIS_HOST=localhost
REDIS_CLIENT=predis
REDIS_PASSWORD=null
REDIS_PORT=6379
REDIS_SCHEME=tcp
REDIS_DB=0
```

Open App\Controllers\BaseController.php

add ` $this->queue = service('queue');` on function initController
```php
//--------------------------------------------------------------------
// Preload any models, libraries, etc, here.
//--------------------------------------------------------------------
// E.g.:
// $this->session = \Config\Services::session();

 $this->queue = service('queue');;
```
## Usage

Example job
```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class SendEmail implements ShouldQueue
{

    use InteractsWithQueue, Queueable, SerializesModels;

    public function fire($e, $payload) {
        $this->onQueue('processing');
        echo "FIRE\n";
        $email = \Config\Services::email();
        $config = [
            'protocol' => getenv('EMAIL_PROTOCOL'),
            'SMTPUser' => getenv('EMAIL_SMTP_USER'),
            'SMTPPass' => getenv('EMAIL_SMTP_PASS'),
            'SMTPHost' => getenv('EMAIL_SMTP_HOST'),
            'SMTPPort' => getenv('EMAIL_SMTP_PORT'),
            'SMTPCrypto' => 'ssl',
            'mailType' => 'html',
        ];
        $email->initialize($config);
        $email->setFrom('email@example.com', 'John Doe');
        $email->setTo($payload['to']);
        $email->setSubject('Hello');
        $email->setMessage('Hello email');
        $email->send();
        
        $e->delete();
    }

}

```

### How to use in controller
```php
<?php 

namespace App\Controllers;

class Home extends BaseController
{
	public function index()
	{
		$this->queue->push('\App\Jobs\SendEmail', ['to' => 'target@example.com']);
	}
}

```

### Run queue worker
```sh
php spark queue:work
```

## More info usefull link docs laravel
- [Queues: Getting Started](https://laravel.com/docs/8.x/queues)


## License

This package is free software distributed under the terms of the [MIT license](LICENSE.md).

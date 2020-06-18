# Binnacle
 practical example of how to generate a log in laravel so that the user registers all the interaction with the system with monolog
 
(ejemplo práctico de cómo generar un registro en laravel para que el usuario registre toda la interacción con el sistema con monolog)

## 1. install monolog: This package will allow you to create the records of each interaction with the user
 
    $ composer require monolog/monolog
 
## 2. create monolog middleware: will allow us to instantiate the log in any system controller
 
    php artisan make:middleware MonologMiddleware
    
## 3. Enter the MonologMiddleware code (MonologMiddleware.php)

```php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Support\Facades\Auth;
use App\Models\User;

use Monolog\Logger;
use Monolog\Handler\StreamHandler;

class MonologMiddleware
{

    public function handle($request, Closure $next)
    {
        $action = app('request')->route()->getAction();
        $controller = class_basename($action['controller']);
        $controller = snake_case($controller);
        list($name2,$action2) = explode('@', $controller);
        list($controller, $action) = explode('_', $controller);
        list($name,$action) = explode('@', $action);

        switch ($action) {
            case 'index':
                $action = 'Index';
                break;
            case 'show':
                $action = 'Ver';
                break;
            case 'store':
                $action = 'Crear';
                break;
            case 'update':
                $action = 'Update';
                break;
            case 'destroy':
                $action = 'Eliminar';
                break;
            default:
            $action = $action2;
            break;

        }

                $user = User::find(\Auth::user()->id);

                $description = "el usuario Nº ". $user->id . " de nombre " . $user->person->name . " realizo esta acción";
    
                $context = [
                    'user_id' => \Auth::user()->id,
                    'operation' => $action,
                    'from' => $controller,
                    'description' => $description,
                    'os' => php_uname($_SERVER['REMOTE_ADDR']),
                    'visitor' => $request->ip(),
                    'device' => $this->returnMacAddress()
                    ];
    
                    $log = new Logger($controller);
                    $log->pushHandler(new StreamHandler(public_path().'/storage/monolog/binnacle.log', Logger::DEBUG));
                    $log->Alert("Nuevo Registro de la Bitacora con las siguiente especificaciones.: ",  $context);

        return $next($request);
    }


    function returnMacAddress() {
        $location = `which arp`;
       $arpTable = `arp -a`;
       $arpSplitted = explode("\\n",$arpTable);
       $remoteIp = getenv('REMOTE_ADDR');
       foreach ($arpSplitted as $value) {
          $valueSplitted = explode(" ",$value);
          foreach ($valueSplitted as $spLine) {
           if (preg_match("/$remoteIp/",$spLine)) {
                $ipFound = true;
          }
        if (isset($ipFound)) {
           reset($valueSplitted);
           foreach ($valueSplitted as $spLine) {
                 if (preg_match("/[0-9a-f][0-9a-f][:-]".
                     "[0-9a-f][0-9a-f][:-]".
                     "[0-9a-f][0-9a-f][:-]".
                    "[0-9a-f][0-9a-f][:-]".
                    "[0-9a-f][0-9a-f][:-]".
                  "[0-9a-f][0-9a-f]/i",$spLine)) {
                     return $spLine;
                  }
              }
         }
        $ipFound = false;
       }
       }
      return false;
      }

}
```

## 4. then instantiate the constructor in any controller of the application
```php

    public function __construct()
    {
        $this->middleware('MonologMiddleware');
    }
    
```
### Example: the registry lets you know what controller is instantiating, the method used, a description of the user id and the name of the same, the visitor's IP, the operating system and the mac address.
 
```txt
 [2020-06-15T01:02:10.181037-05:00] user.ALERT: Nuevo Registro de la Bitacora con las siguiente especificaciones.:  {"user_id":2,"operation":"user_settings","from":"user","description":"el usuario Nº 2 de nombre Isabel Sánchez realizo esta acción","os":"Windows NT PERFILES 10.0 build 18362 (Windows 10) AMD64","visitor":"127.0.0.1","device":"60-33-26-90-5e-39"} []
```


## Recommendation: to take advantage of this registry when it comes to obtaining statistics of which module the user visits the most and applying it to graphics for decision-making can create a migration where they save all the information to generate very interesting queries and present it in a management module.
 
 

<?php

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

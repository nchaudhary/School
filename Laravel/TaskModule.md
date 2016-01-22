##Database Migrations

###new database migration for our tasks table

```
php artisan make:migration create_tasks_table --create=tasks

php artisan migrate

```

##Eloquent Models
```
php artisan make:model Task

```
Edit Model file Task.php
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Task extends Model
{
   /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['name'];
    use SoftDeletes;

 /**
     * Get the user that owns the task.
     */
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

##Routing
```
Route::group(['middleware' => 'web'], function () {
//Auth Routs
    Route::auth();
    
    Route::get('/home', 'HomeController@index');
    
  //Task routs
    Route::get('/tasks', 'TaskController@index');
    Route::get('/tasks/trash', 'TaskController@trashTasks');
  	Route::post('/task', 'TaskController@store');
  	Route::delete('/task/{task}', 'TaskController@destroy');
  	Route::delete('/tasks/trash/{task}', 'TaskController@hardDelete');
  	Route::get('/tasks/edit/{task}', 'TaskController@viewTask');
  	Route::put('/tasks/edit/{task}', 'TaskController@edit');
  	Route::post('/tasks/restore/{task}', 'TaskController@restoreTask');

});
```
## TaskController.php
```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Task;
use App\Http\Requests;
use App\Http\Controllers\Controller;

class TaskController extends Controller
{
    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth');
    }
	/**
	 * Display a list of all of the user's task.
	 *
	 * @param  Request  $request
	 * @return Response
	 */
	public function index(Request $request)
	{
		
	     $tasks = Task::where('user_id', $request->user()->id)->get();
		    return view('tasks.index', [
		        'tasks' => $tasks,
		    ]);
	}
	
	/**
	 * Create a new task.
	 *
	 * @param  Request  $request
	 * @return Response
	 */
	public function store(Request $request)
	{

	    $this->validate($request, [
	        'name' => 'required|max:255',
	    ]);

	    $request->user()->tasks()->create([
	        'name' => $request->name,
	    ]);

	    return redirect('/tasks');
	}
	/**
	 * Destroy the given task.
	 *
	 * @param  Request  $request
	 * @param  Task  $task
	 * @return Response
	 */
	public function destroy(Request $request, Task $task)
	{


	    $task->delete();

	    return redirect('/tasks');
	}
	/**
	 * Perment delete of task for database table.
	 *
	 * @param  Request  $request
	 * @param  Task  $task
	 * @return Response
	 */
	public function hardDelete(Request $request)
	{

	    $tasksTrash=Task::where('id',$request->taskId)->forceDelete() ;
	    return redirect('/tasks');
	}
	/**
	 * Restore soft delete task.
	 *
	 * @param  Request  $request
	 * @param  Task  $task
	 * @return Response
	 */
	public function restoreTask(Request $request)
	{

	    $tasksTrash=Task::where('id',$request->taskId)->restore() ;
	    return redirect('/tasks');
	}

	/**
	 * Update existing task.
	 *
	 * @param  Request  $request
	 * @param  Task  $task
	 * @return Response
	 */
	public function viewTask(Request $request, Task $task)
	{
		 return view('tasks.edit', [
		        'task' => $task,
		    ]);
	}

	/**
	 * Update existing task.
	 *
	 * @param  Request  $request
	 * @param  Task  $task
	 * @return Response
	 */
	public function edit(Request $request, Task $task)
	{
	
		$task = Task::find($task->id);

		$task->name = $request->name;

		$task->save();

	    return redirect('/tasks');
	}
	/**
	 * Trashed tasks list.
	 *
	 * @param  Request  $request
	 * @param  Task  $task
	 * @return Response
	 */
	public function trashTasks(Request $request)
	{
	
		$tasksTrash=\App\User::find(\Auth::id())->tasks()->onlyTrashed()->get();
		
		    return view('tasks.trash', [
		        'tasksTrash' => $tasksTrash,
		    ]);
	}
}

```


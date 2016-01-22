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
##Task Module View Files :
###edit.blade.php
```
@extends('layouts.app')
@section('title', 'Edit Tasks')
@section('content')
<div class="container">
    <div class="row">
    <!-- Bootstrap Boilerplate... -->
 <div class="col-md-10 col-md-offset-1">
            <div class="panel panel-default">
                <div class="panel-heading">Edit Task</div>
    <div class="panel-body">
       
        <!-- New Task Form -->
        <form action="{{ url('tasks/edit').'/'.$task->id }}" method="POST" class="form-horizontal">
            {{ csrf_field() }}
             {{ method_field('PUT') }}
            <!-- Task Name -->
            <div class="form-group">
                <label for="task-name" class="col-sm-3 control-label">Task</label>

                <div class="col-sm-6">
                    <input type="text" name="name" id="task-name" value="{{ $task->name}}" class="form-control">
                </div>
            </div>

            <!-- Add Task Button -->
            <div class="form-group">
                <div class="col-sm-offset-3 col-sm-6">
                    <button type="submit" class="btn btn-default">
                        <i class="fa"></i> Update Task
                    </button>
                </div>
            </div>
        </form>
    </div>
</div>
</div>
</div>
    
 </div>
</div>
@stop
```
###index.blade.php
```
@extends('layouts.app')
@section('title', 'Tsak Add')
@section('content')
<div class="container">
    <div class="row">
    <!-- Bootstrap Boilerplate... -->
    <div class="col-md-10 col-md-offset-1">
            <div class="panel panel-default">
                <div class="panel-heading">Add Task - (<a href="/tasks/trash">Trash List</a>)</div>
    <div class="panel-body">
       
        <!-- New Task Form -->
        <form action="{{ url('task') }}" method="POST" class="form-horizontal">
            {{ csrf_field() }}

            <!-- Task Name -->
            <div class="form-group">
                <label for="task-name" class="col-sm-3 control-label">Task</label>

                <div class="col-sm-6">
                    <input type="text" name="name" id="task-name" class="form-control">
                </div>
            </div>

            <!-- Add Task Button -->
            <div class="form-group">
                <div class="col-sm-offset-3 col-sm-6">
                    <button type="submit" class="btn btn-default">
                        <i class="fa fa-plus"></i> Add Task
                    </button>
                </div>
            </div>
        </form>
    </div>
</div>
</div>
</div>

    <!-- TODO: Current Tasks -->
     <!-- Current Tasks -->
    @if (count($tasks) > 0)
        <div class="panel panel-default">
            <div class="panel-heading">
                Current Tasks
            </div>

            <div class="panel-body">
                <table class="table table-striped task-table">

                    <!-- Table Headings -->
                    <thead>
                        <th>Task</th>
                        <th>Delete Action</th>
                        <th>Edit Action</th>
                    </thead>

                    <!-- Table Body -->
                    <tbody>
                        @foreach ($tasks as $task)
                            
                            <tr>
                                <!-- Task Name -->
                                <td class="table-text">
                                    <div>{{ $task->name }}</div>
                                </td>

                                <!-- Delete Button -->
                                <td>
                                    <form action="{{ url('task/'.$task->id) }}" method="POST">
                                        {{ csrf_field() }}
                                        {{ method_field('DELETE') }}

                                        <button onclick="return ConfirmDelete();">Delete Task</button>
                                    </form>
                                </td>
                                <td>
                                    <a href="{{ url('tasks/edit/'.$task->id) }}"> Edit Task</a>
                                </td>
                            </tr>
                        @endforeach
                    </tbody>
                </table>
            </div>
        </div>
    
    @endif
 </div>
</div>
@stop
<script>

  function ConfirmDelete()
  {
  var x = confirm("Are you sure you want to delete?");
  if (x)
    return true;
  else
    return false;
  }

</script>
```
###trash.blade.php
```
@extends('layouts.app')
@section('title', 'Tsak Add')
@section('content')
<div class="container">
    <div class="row">
    <!-- Bootstrap Boilerplate... -->
    <div class="col-md-10 col-md-offset-1">
            <div class="panel panel-default">
                <div class="panel-heading">Trash Task List</div>
    <div class="panel-body">
    <table class="table table-striped task-table">

                    <!-- Table Headings -->
                    <thead>
                        <th>Task</th>
                        <th>&nbsp;</th>
                    </thead>

                    <!-- Table Body -->
                    <tbody>
                        @foreach ($tasksTrash as $task)
                            
                            <tr>
                                <!-- Task Name -->
                                <td class="table-text">
                                    <div>{{ $task->name }}</div>
                                </td>

                                <!-- Delete Button -->
                                <td>
                                    <form action="{{ url('tasks/trash/'.$task->id) }}" method="POST">
                                        {{ csrf_field() }}
                                        {{ method_field('DELETE') }}
                                        <input name="taskId" type="hidden" value="{{$task->id}}">
                                        <button onclick="return ConfirmDelete();">Delete Permanent</button>
                                    </form>
                                </td>
                                <td>
	                                <form action="{{ url('tasks/restore/'.$task->id) }}" method="POST">
	                                        {{ csrf_field() }}
	                                      {{ method_field('POST') }}
	                                        <input name="taskId" type="hidden" value="{{$task->id}}">
	                                       <button> Restore Task</a>
	                                    </form>
                                    
                                </td>
                                
                            </tr>
                        @endforeach
                    </tbody>
                </table>
            </div>
        </div>
          </div>
        </div>
        @stop
<script>

  function ConfirmDelete()
  {
  var x = confirm("Are you sure you want to delete?");
  if (x)
    return true;
  else
    return false;
  }

</script>
```




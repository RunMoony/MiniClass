# 안드로메다(문광제 사원, 백현수 사원, 송민정 사원)

# Todo List
#### 개인프로젝트, Kotlin, Android Studio
#### 개발기간 : 2023.06.01 ~ 2023.06.30

## 애플리케이션 설명
간단히 할 일을 추가할 수 있는 Todo List 애플리케이션입니다.

#### TodoList 애플리케이션을 만든 이유
새로운 언어를 습득하기 위한 가장 좋은 토이프로젝트가 TodoList 라 생각해서 구현하였습니다.

## 순서
- 1. 사용한 개념 및 기능
- 2. 느낀점

#### 추가하기<br>

하단의 EditText에 내용을 입력 후 '추가' Button을 누르면 viewModel객체 내의 MutableLiveData의 value의 값이 변경되도록 설정했습니다.
    
#### 완료하기<br>


CheckBox 체크 시 isChecked의 값을 todo.isDone 넣어주고, todo.isDone의 값에 따라 취소선(-)이 표시되게 하였습니다.

'X'버튼을 통해 삭제하거나 menu의 완료 삭제를 통해 삭제를 합니다.
todoDao의 delete 메서드를 통해 삭제하는 방식으로 진행하였습니다.

#### 검색하기<br>

<pre><code>
TodoDao.kt
    //Query문을 이용해 키워드로 todoList 얻기.
    @Query("select * from todo where text like '%' ||:text || '%' order by date,time desc")
    fun getTodosByText(text: String) : MutableList<Todo>
    </code></pre>
Room의 Query어노테이션의 속성에 query문을 넣어주었고, getTodosByText의 매개변수로 들어온 text를 이용하여 검색하고
리스트를 받는 방식으로 진행하였습니다.


<pre><code>MainActivity.kt
    override fun onCreateOptionsMenu(menu: Menu?): Boolean {
         ...
         ..
         .
      //검색 기능
        searchView.setOnQueryTextListener(object : SearchView.OnQueryTextListener {
            override fun onQueryTextSubmit(query: String?): Boolean {
                //검색어 입력버튼을 누른 후 내용이 있다면
                if (query != "" && query != null) {
                    //검색어를 통해 TodoList를 얻어옴.
                    todoList.value = viewModel.getTodosByText(query)
                } else {
                    setList()
                }
                return true
            }
            override fun onQueryTextChange(newText: String?): Boolean {
                return false
            }
        })

        //검색 닫기를 누른 후 기본 TodoList로 복귀
       searchView.setOnCloseListener {
            setList()
            false
        }
        return super.onCreateOptionsMenu(menu)
    }
</code></pre>


#### 정렬하기<br>

메뉴의 등록일 순 얻기는 Todo객체의 PrimaryKey인 registerTime: Long 객체로 Todo객체가 생성될 때</br>
System.currentTimeMills()를 얻습니다.</br>
그것을 기준으로 등록일 순 정렬을 얻었습니다.</br>

날짜순 얻기는 Todo객체의 date와 time을 통해 검색됩니다.
<pre><code>TodoDao.kt
    @Query("select * from Todo order by registerTime desc")
    fun getAll() : MutableList<Todo>

    //전체 얻기. 날짜순으로 얻기.
    @Query("select * from todo order by date, time desc")
    fun getAllTimeOrder() : MutableList<Todo>
</code></pre>



### Room, LiveData, ViewModel
<pre>
Room, LiveData, ViewModel을 이용한 MVVM패턴을 활용하여
MutableLiveData<MutableList<Todo>>타입으로 받고, 
Todo객체를 item_list의 View에 바인딩 해줌으로써
화면에 목록으로써 구현됨.
</pre>
 *Room
  - Entity 생성
  <pre><code>@Entity
data class Todo(var text: String?, var isDone: Boolean = false) : Serializable {
    // 등록하는 순간 System의 시간을 받으므로 등록일 순 정렬에 사용할 기준이 됨.
    @PrimaryKey
    var registerTime : Long = System.currentTimeMillis()

    @ColumnInfo
    var hashTag: String? = null

    // 날짜 기준 검색의 기준이 됨.
    @ColumnInfo
    var time: String? = null
    @ColumnInfo
    var date: String? = null
    @ColumnInfo
    var dateLong: Long? = null
    ...
    ..
    .
</code></pre>
  - Dao 생성
 <pre><code>
 @Dao
interface TodoDao {
    //전체 얻기. 기본값. 등록순서로 보이기
    @Query("select * from Todo order by registerTime desc")
    fun getAll() : MutableList<Todo>

    //전체 얻기. 날짜순으로 얻기.
    @Query("select * from todo order by date, time desc")
    fun getAllTimeOrder() : MutableList<Todo>

    //할일 키워드로 얻기
    @Query("select * from todo where text like '%' ||:text || '%' order by date,time desc")
    fun getTodosByText(text: String) : MutableList<Todo>
    ....
    ...
    ..
    .
</code></pre>

 - Database 생성
 <pre><code>
 @Database(entities = [Todo::class], version = 2)
abstract class AppDatabase : RoomDatabase() {
    abstract fun todoDao(): TodoDao
}
</code></pre>
  
 2. ViewModel
 <pre><code>
 // TodoViewModel 객체를 생성할 때 Application을 생성자에 넣어줘야하기때문에,
// 생성자가 있는 ViewModel을 생성하기 위해서는 내가 만들고자 하는 viewModel을 AndroidViewModel(application)을 상속받아 생성하
// ViewModel을 상속받은 경우는 ViewModelProvider.Factory 인터페이스를 구현한 클래스를 이용해서 생성해야함.


class ViewModelProviderFactory(val context: Context) : ViewModelProvider.Factory{
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        return if (modelClass.isAssignableFrom(TodoViewModel::class.java)) {
            TodoViewModel(context) as T
        } else {
            throw IllegalArgumentException()
        }
    }
}</code></pre>


3. LiveData
TodoDao객체에서 todoList 검색 결과를 받은 후 MutableLiveData<List<Todo>>객체 생성 시 인자로 해당 todoList를 넣어주어 생성
  <pre><code>val mutableLiveData = MutableLiveData<MutableList<Todo>>(todos)
</code></pre>
 <pre><code>    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //뷰모델 받아오기
        viewModel = ViewModelProvider(this, ViewModelProviderFactory(this.application))
            .get(TodoViewModel::class.java)

        //recycler view에 보여질 아이템 Room에서 받아오기
        todoList = viewModel.mutableLiveData
        todoList.observe(this, Observer {
            todayAdapter.itemList = it
            todayAdapter.notifyDataSetChanged()
        })

</code></pre>
 
 


### 마치며
코틀린을 배우고 프로젝트까지 직접해보면서, 개발역량을 키울수있었습니다.
개발역량을 꾸준히 향상시켜서 교보생명 시스템을 효율적으로 개선,개발해나가겠습니다. 

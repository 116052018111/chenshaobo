
## 时间戳
### 关键代码
##### 在noteslist_item.xml中加入一个TextView
```
<TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"        
        android:layout_height="wrap_content"        
        android:paddingLeft="5dip"        
        android:singleLine="true"       
        android:gravity="center_vertical"       
    />
```
##### 修改时间戳类型
```
long now = System.currentTimeMillis();
Date date = new Date(now);
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
String dateFormat = simpleDateFormat.format(date);
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
```
##### 在NotesList中，为PROJECTION添加时间修改
```
NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
```
##### 加入时间修改
```
int[] viewIDs = { android.R.id.text1, android.R.id.text2 }
```
### 实验截图
![image](https://github.com/116052018111/chenshaobo/blob/master/QQ20201220014232.png)
![image](https://github.com/116052018111/chenshaobo/blob/master/QQ20201220014328.png)

## 笔记查询
### 关键代码
##### 添加搜索图标
```
<item
        android:id="@+id/menu_search"
        android:icon="@android:drawable/ic_search_category_default"
        android:showAsAction="always"
        android:title="search">
    </item>
```
##### 添加menu_search图标的点击选择事件
```
case R.id.menu_search:
                Intent intent = new Intent();
                intent.setClass(this, NoteSearch.class);
                this.startActivity(intent);
                return true;
```
##### 添加note_search.xml活动
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="搜索">
    </SearchView>
    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        >
    </ListView>
</LinearLayout>
```
##### 添加NoteSearch活动
```
public class NoteSearch extends Activity implements SearchView.OnQueryTextListener{
    ListView listview;//
    SQLiteDatabase sqLiteDatabase;
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE // 修改时间
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,WindowManager.LayoutParams.FLAG_FULLSCREEN);//设置全屏显示
        super.setContentView(R.layout.note_search);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        listview= (ListView) findViewById(R.id.list_view);//获取listview
        sqLiteDatabase=new NotePadProvider.DatabaseHelper(this).getReadableDatabase();//对数据库进行操作
        SearchView search= (SearchView) findViewById(R.id.search_view);//获取搜索视图
        search.setOnQueryTextListener(NoteSearch.this);
        listview.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {//为每个item添加点击事件，点击可以查看笔记具体内容
                Uri uri = ContentUris.withAppendedId(getIntent().getData(), l);
                String action = getIntent().getAction();
                if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
                    setResult(RESULT_OK, new Intent().setData(uri));
                } else {
                    startActivity(new Intent(Intent.ACTION_EDIT, uri));
                }
            }
        });
    }
    @Override
    public boolean onQueryTextSubmit(String query) {
        return true;
    }
    @Override
    public boolean onQueryTextChange(String newText) {//实现模糊查询，通过标题或者内容进行查询
        Cursor cursor=sqLiteDatabase.query(
                NotePad.Notes.TABLE_NAME,
                PROJECTION,
                NotePad.Notes.COLUMN_NAME_TITLE+" like ? ",
                new String[]{"%"+newText+"%","%"+newText+"%"},
                null,
                null,
                NotePad.Notes.DEFAULT_SORT_ORDER);
        int[] viewIDs = { android.R.id.text1,android.R.id.text2};
        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                NoteSearch.this,                             
                R.layout.noteslist_item.xml,          
                cursor,                           
                new String[]{NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE},
                viewIDs
        );
        listview.setAdapter(adapter);
        return true;
    }
}
```
### 实验截图
![image](https://github.com/116052018111/chenshaobo/blob/master/QQ20201220212453.png)
![image](https://github.com/116052018111/chenshaobo/blob/master/QQ20201220212508.png)
![image](https://github.com/116052018111/chenshaobo/blob/master/QQ20201220212613.png)

## UI界面美化and更换背景颜色
### 关键代码
##### NotePad.java 中设置颜色编号
```
public static final String COLUMN_NAME_BACK_COLOR = "color";
public static final int DEFAULT_COLOR = 0;
public static final int colorAccent_COLOR = 1;
public static final int violet_COLOR = 2;
public static final int GREEN_COLOR = 3;
public static final int RED_COLOR = 4; 
```
##### 在NotePadProvider.java中修改创建数据库的语句
```
NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER"
```
##### 实例化和设置静态对象的块
```
static{
    sNotesProjectionMap.put(
        NotePad.Notes.COLUMN_NAME_BACK_COLOR,
        NotePad.Notes.COLUMN_NAME_BACK_COLOR);
}
```
##### 增加创建新笔记时需要执行的语句，给每条笔记设置默认背景颜色
```
if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
}
```
##### 在NoteList和NoteSearch中的PROJECTION添加color属性
```
NotePad.Notes.COLUMN_NAME_BACK_COLOR,
```
##### 使用bindView将颜色填充到ListView。新建一个MyCursorAdapter继承SimpleCursorAdapter
```
public class MyCursorAdapter extends SimpleCursorAdapter {
    public MyCursorAdapter(Context context, int layout, Cursor c,
                        String[] from, int[] to) {
        super(context, layout, c, from, to);
    }
    @Override
    public void bindView(View view, Context context, Cursor cursor){
        super.bindView(view, context, cursor);
        int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
            case NotePad.Notes.YELLOW_COLOR:
                view.setBackgroundColor(Color.rgb(247, 216, 133));
                break;
            case NotePad.Notes.BLUE_COLOR:
                view.setBackgroundColor(Color.rgb(165, 202, 237));
                break;
            case NotePad.Notes.GREEN_COLOR:
                view.setBackgroundColor(Color.rgb(161, 214, 174));
                break;
            case NotePad.Notes.RED_COLOR:
                view.setBackgroundColor(Color.rgb(244, 149, 133));
                break;
            default:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
        }
    }
}
```
##### 将NoteList和NoteSearch里的适配器改为MyCursorAdapter
```
MyCursorAdapter adapter
    = new MyCursorAdapter(
              this,
              R.layout.noteslist_item,
              cursor, 
              dataColumns,
              viewIDs
      );
```
##### 新建colors.xml和color_select.xml
```
<resources>
    <color name="color1">#be96df</color>
</resources>
```
```
<selector xmlns:android="http://schemas.android.com/apk/res/android">
<item android:drawable="@color/color1"
    android:state_pressed="true"/>
</selector>
```
##### 在notelist_item.xml里为控件添加选择器
```
android:background="@drawable/color_select"
```
### 实验截图
![image](https://github.com/116052018111/chenshaobo/blob/master/QQ20201221012418.png)
![image](https://github.com/116052018111/chenshaobo/blob/master/QQ20201221012706.png)

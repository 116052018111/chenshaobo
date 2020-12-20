# chenshaobo
## 时间戳
### 代码
#### 在noteslist_item.xml中加入一个TextView
'''<TextView'
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingLeft="5dip"
        android:singleLine="true"
        android:gravity="center_vertical"
    />
'''
#### 修改时间戳类型
long now = System.currentTimeMillis();
Date date = new Date(now);
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
String dateFormat = simpleDateFormat.format(date);
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
#### 在NotesList中，为PROJECTION添加时间修改
NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
#### 加入时间修改
int[] viewIDs = { android.R.id.text1, android.R.id.text2 }
### 实验截图


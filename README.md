# NotePad
This is an AndroidStudio rebuild of google SDK sample NotePad
一.增加时间戳：
res—layout—notelist_item.xml布局文件中的代码改为：
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content">
    >
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"
        />

    <TextView
        android:id="@android:id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dp"
        android:singleLine="true" />
</LinearLayout>
在NoteList类的PROJECTION中添加COLUMN_NAME_MODIFICATION_DATE字段：
/**
 * The columns needed by the cursor adapter
 */
private static final String[] PROJECTION = new String[] {
        NotePad.Notes._ID, // 0
        NotePad.Notes.COLUMN_NAME_TITLE, // 1
        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//此为添加代码
};

在NoteList类增加dataColumns中装配到ListView的内容：
需将代码：
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE } ;
int[] viewIDs = { android.R.id.text1 };
改为：
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,
        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE  } ;
int[] viewIDs = { android.R.id.text1 ,android.R.id.text2};
在NoteEditor类的updateNote方法中获取当前系统的时间，并对时间进行格式化：
将代码：
ContentValues values = new ContentValues();
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, System.currentTimeMillis());
改为：
ContentValues values = new ContentValues();
Long now = Long.valueOf(System.currentTimeMillis());
SimpleDateFormat sf = new SimpleDateFormat("yy/MM/dd HH:mm");
Date d = new Date(now);
String format = sf.format(d);
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, format);
效果展示：
![Screenshot_2021-11-28-22-16-31-410_com example my](https://user-images.githubusercontent.com/90604327/143771880-de8cfa33-43cf-4a89-8404-b86db80d006c.jpg)
二.添加搜索框
在res—values—strings.xml中添加menu_search字段
在res—menu—list_options_menu.xml布局文件中添加搜索功能，新增menu_search：
<item
    android:id="@+id/menu_search"
    android:icon="@android:drawable/ic_menu_search"
    android:title="@string/menu_search"
    android:showAsAction="always" />
在res—layout中新建一个查找笔记内容的布局文件note_search.xml：
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        />
    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        />
</LinearLayout>
在NoteList类中的onOptionsItemSelected方法中新增search查询的处理：
case R.id.menu_search:
    //查找功能
    //startActivity(new Intent(Intent.ACTION_SEARCH, getIntent().getData()));
    Intent intent = new Intent(this, NoteSearch.class);
    this.startActivity(intent);
    return true;
新建一个NoteSearch类用于search功能的功能实现：
package com.example.android.notepad;
import android.app.Activity;
import android.content.Intent;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.widget.ListView;
import android.widget.SearchView;
import android.widget.SimpleCursorAdapter;
import android.widget.Toast;

public class NoteSearch extends Activity implements SearchView.OnQueryTextListener
{
    ListView listView;
    SQLiteDatabase sqLiteDatabase;
    /**
     * The columns needed by the cursor adapter
     */
    private static final String[] PROJECTION = new String[]{
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE//时间
    };

    public boolean onQueryTextSubmit(String query) {
        Toast.makeText(this, "您选择的是："+query, Toast.LENGTH_SHORT).show();
        return false;
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search);
        SearchView searchView = findViewById(R.id.search_view);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        listView = findViewById(R.id.list_view);
        sqLiteDatabase = new NotePadProvider.DatabaseHelper(this).getReadableDatabase();
        //设置该SearchView显示搜索按钮
        searchView.setSubmitButtonEnabled(true);

        //设置该SearchView内默认显示的提示文本
        searchView.setQueryHint("查找");
        searchView.setOnQueryTextListener(this);

    }
    public boolean onQueryTextChange(String string) {
        String selection1 = NotePad.Notes.COLUMN_NAME_TITLE+" like ? or "+NotePad.Notes.COLUMN_NAME_NOTE+" like ?";
        String[] selection2 = {"%"+string+"%","%"+string+"%"};
        Cursor cursor = sqLiteDatabase.query(
                NotePad.Notes.TABLE_NAME,
                PROJECTION, // The columns to return from the query
                selection1, // The columns for the where clause
                selection2, // The values for the where clause
                null,          // don't group the rows
                null,          // don't filter by row groups
                NotePad.Notes.DEFAULT_SORT_ORDER // The sort order
        );
        // The names of the cursor columns to display in the view, initialized to the title column
        String[] dataColumns = {
                NotePad.Notes.COLUMN_NAME_TITLE,
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
        } ;
        // The view IDs that will display the cursor columns, initialized to the TextView in
        // noteslist_item.xml
        int[] viewIDs = {
                android.R.id.text1,
                android.R.id.text2
        };
        // Creates the backing adapter for the ListView.
        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                this,                             // The Context for the ListView
                R.layout.noteslist_item,         // Points to the XML for a list item
                cursor,                           // The cursor to get items from
                dataColumns,
                viewIDs
        );
        // Sets the ListView's adapter to be the cursor adapter that was just created.
        listView.setAdapter(adapter);
        return true;
    }
}
效果展示（右上角）：
![Screenshot_2021-11-28-22-16-31-410_com example my](https://user-images.githubusercontent.com/90604327/143772203-e8005465-12a7-4c14-bfa6-dafe402af698.jpg)
三.增加背景图：
将图片文件添加到drawable里，更名为bk
在note_editor.xml中添加代码：
android:background="@drawable/bk"
在notelist_item.xml中添加代码：
android:background="@drawable/bk"
运行效果：![Screenshot_2021-11-28-22-16-31-410_com example my](https://user-images.githubusercontent.com/90604327/143772505-802bea55-4dd1-4a00-b32b-973924113d4e.jpg)
四.改变字体颜色大小：
在NoteEditor中的onOptionsItemSelected中添加代码：
        case R.id.font_10:
            mText.setTextSize(20);
            Toast toast=Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
            toast.show();
            break;
        case R.id.font_16:
            mText.setTextSize(32);
            Toast toast2=Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
            toast2.show();
            break;
        case R.id.font_20:
            mText.setTextSize(40);
            Toast toast3=Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
            toast3.show();
            break;
        case R.id.red_font:
            mText.setTextColor(Color.RED);
            Toast toast4=Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
            toast4.show();
            break;
        case R.id.yellow_font:
            mText.setTextColor(Color.YELLOW);
            Toast toast5=Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
            toast5.show();
            break;

        }
        return super.onOptionsItemSelected(item);
在strings.xml中添加代码：
    <string name="font_size">字体大小</string>
    <string name="font10">10号字</string>
    <string name="font16">16号字</string>
    <string name="font20">20号字</string>
    <string name="font_color">字体颜色</string>
    <string name="red_title">红色</string>
    <string name="yellow_title">黄色</string>
在editor_options_menu.xml中添加代码：
<item
        android:id="@+id/font_size"
        android:title="@string/font_size">
        <!--子菜单-->
        <menu>
            <!--定义一组单选菜单项-->
            <group>
                <!--定义多个菜单项-->
                <item
                    android:id="@+id/font_10"
                    android:title="@string/font10"
                    />

                <item
                    android:id="@+id/font_16"
                    android:title="@string/font16" />
                <item
                    android:id="@+id/font_20"
                    android:title="@string/font20" />
            </group>
        </menu>
    </item>

    <item
        android:title="@string/font_color"
        android:id="@+id/font_color"
        >
        <menu>
            <!--定义一组普通菜单项-->
            <group>
                <!--定义两个菜单项-->
                <item
                    android:id="@+id/red_font"
                    android:title="@string/red_title" />
                <item
                    android:title="@string/yellow_title"
                    android:id="@+id/yellow_font"/>
            </group>
        </menu>
    </item>
运行效果：![Screenshot_2021-11-28-22-16-35-233_com example my](https://user-images.githubusercontent.com/90604327/143772747-babf0ab0-0b33-498c-8c6e-41394491c3a7.jpg)
![Screenshot_2021-11-28-22-16-38-134_com example my](https://user-images.githubusercontent.com/90604327/143772759-4f507a10-1adc-476d-aa3e-b1625af373b8.jpg)
![Screenshot_2021-11-28-22-16-46-439_com example my](https://user-images.githubusercontent.com/90604327/143772764-57f29beb-fcc7-447d-8e0d-2f8a6d2d96a4.jpg)


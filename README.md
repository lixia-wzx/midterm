  NotePad实现四种功能：

​       一：实现显示笔记时间戳功能

​       二： 实现记事本搜索功能

​       三： 实现笔记分类功能

​       四： UI美化



一： 实现显示笔记时间戳功能

​           （1） 更改NoteList布局文件，添加TextView来显示时间戳  

​                            notelist_item.xml 代码：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="?android:attr/listPreferredItemHeight"
    android:orientation="vertical">
    
    <!-- 
       这里显示笔记标题
     -->
    <TextView
        android:id="@+id/text1"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="10dip"
        android:layout_weight="1"
        android:layout_margin="0dp"
        />
    <!-- 
     这里显示笔记时间戳
   -->
    <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:textSize="12dp"
        android:gravity="center_vertical"
        android:paddingLeft="10dip"
        android:layout_weight="1"
        android:layout_margin="0dp"
        />
</LinearLayout>
```

​             (2） 更改NoteEditor.java 中的UpdateNote方法将当前时间格式化并写入数据库中

```java
    private final void updateNote(String text, String title) {
        //将时间格式化
        Date nowTime = new Date(System.currentTimeMillis());
        SimpleDateFormat sdFormatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String retStrFormatNowDate = sdFormatter.format(nowTime);
        
        ContentValues values = new ContentValues();
        //将时间写入数据库中
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, retStrFormatNowDate);
        values.put(NotePad.Notes.COLUMN_TAG_SELECTION_INDEX,items[checkItem]);
        values.put(NotePad.Notes.COLUMN_BACKGROUND_COLOR,colorBack);
        values.put(NotePad.Notes.COLUMN_TEXT_COLOR,colorText);
        values.put(NotePad.Notes.COLUMN_TEXT_NOTIFICATION_DATE,date+time);
        if (mState == STATE_INSERT) {
            if (title == null) {
                int length = text.length();
                title = text.substring(0, Math.min(5, length));
                if (length > 30) {
                    int lastSpace = title.lastIndexOf(' ');
                    if (lastSpace > 0) {
                        title = title.substring(0, lastSpace);
                    }
                }
            }
            values.put(NotePad.Notes.COLUMN_NAME_TITLE, title);

        } else if (title != null) {
            values.put(NotePad.Notes.COLUMN_NAME_TITLE, title);
        }
        values.put(NotePad.Notes.COLUMN_NAME_NOTE, text);
        getContentResolver().update(
                mUri,    // The URI for the record to update.
                values,  // The map of column names and new values to apply to them.
                null,    // No selection criteria are used, so no where columns are necessary.
                null     // No where columns are used, so no where arguments are necessary.
        );

    }
```

​           (3）在NoteList.java中的PROJECTION数组中增加时间字段，在dataColumns和ViewIDs数组中增加时间字段

```java
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //新增的时间戳字段
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
    };
```

```java
private String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;
```

```java
private int[] viewIDs = { R.id.text1,R.id.text2 };
```

效果截图：

![image-20201220142040524](https://github.com/wangwang01-wzx/midterm/blob/main/image-20201220142040524.png)

二： 实现记事本搜索功能

​     （1）修改NoteList.java布局文件   Listview.xml实现搜索框

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/dl"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:focusableInTouchMode="true"
        android:focusable="true"
       >
        <android.support.v7.widget.Toolbar
            android:id="@+id/tb"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="@color/colorPrimary"
            app:titleTextColor="@color/white"
            app:title="NotePad"
            app:navigationIcon="?attr/homeAsUpIndicator"
            >
        </android.support.v7.widget.Toolbar>
        <android.support.v7.widget.SearchView
            android:id="@+id/sv"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            >
            
            <!-- 搜索框 -->
        </android.support.v7.widget.SearchView>
        <android.support.design.widget.CoordinatorLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content">
            <ListView
                android:id="@+id/tv"
                android:layout_width="match_parent"
                android:layout_height="527dp"/>

        </android.support.design.widget.CoordinatorLayout>
    </LinearLayout>
    <android.support.design.widget.NavigationView
        android:id="@+id/nv"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:fitsSystemWindows="true"
        android:layout_gravity="left"
        app:headerLayout="@layout/nv_header"
        app:menu="@menu/nv_menu"
        >
    </android.support.design.widget.NavigationView>
</android.support.v4.widget.DrawerLayout>
```

​        （2）在NoteList.java中实现搜索的方法

```java
    private void SearchView(){
        searchView=findViewById(R.id.sv);
        searchView.onActionViewExpanded();
        searchView.setQueryHint("搜索");
        searchView.setSubmitButtonEnabled(true);
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String s) {
                return false;
            }
            @Override
            public boolean onQueryTextChange(String s) {
                if(!s.equals("")){
                    String selection=NotePad.Notes.COLUMN_NAME_TITLE+" GLOB '*"+s+"*'";
                    updatecursor = getContentResolver().query(
                            getIntent().getData(),          
                            PROJECTION,                    
                            selection,                           
                            null,                            
                            NotePad.Notes.DEFAULT_SORT_ORDER  
                    );
                    if(updatecursor.moveToNext())
                        Log.i("daawdwad",selection);
                }
               else {
                    updatecursor = getContentResolver().query(
                            getIntent().getData(),           
                            PROJECTION,                      
                            null,                        
                            null,                           
                            NotePad.Notes.DEFAULT_SORT_ORDER
                    );
                }
                adapter.swapCursor(updatecursor);
                return false;
            }
        });
    }
```

​      效果截图：

![image-20201220142926139](https://github.com/wangwang01-wzx/midterm/blob/main/image-20201220142926139.png)

![image-20201220143001157](https://github.com/wangwang01-wzx/midterm/blob/main/image-20201220143001157.png)

三： 实现笔记分类功能

​     （1）在数据库中添加标签字段以将记事本分类   

​                   在NotePad.java中添加标签字段

```java
//标签
public static final String COLUMN_TAG_SELECTION_INDEX="tag";
```

​         （2） 在NoteEditor.java布局文件增加选择标签按钮

```xml
<?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
    
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal">
            
            <TextView
                android:id="@+id/etv"
                android:layout_width="wrap_content"
                android:layout_height="match_parent"
                android:layout_weight="2"
                android:textSize="20dp"
                android:layout_marginStart="10dp"
                android:gravity="center_vertical"
                android:text="date"/>
            
            <Button
                android:id="@+id/eb"
                android:layout_width="10dp"
                android:layout_height="match_parent"
                android:layout_weight="1"
                android:background="@color/aqua"
                android:onClick="tagSelect" />
        </LinearLayout>
    
        <Button
            android:id="@+id/dateButtom"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textSize="20dp"
            android:gravity="center_horizontal"
            android:onClick="dateClick"/>

        <view class="com.example.mynotepad.NoteEditor$LinedEditText"
            xmlns:android="http://schemas.android.com/apk/res/android"
            android:id="@+id/note"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="@android:color/transparent"
            android:padding="5dp"
            android:scrollbars="vertical"
            android:fadingEdge="vertical"
            android:gravity="top"
            android:textSize="22sp"
            android:inputType="textCapSentences|textMultiLine"
            />
    </LinearLayout>
```

​            (3)在NoteEditor.java中实现点击按钮弹出AlertDialog对话框，并加入到数据库中

```java
    public void tagSelect(View v){
        AlertDialog.Builder tagbuilder;
        AlertDialog alertDialog;
        tagbuilder=new AlertDialog.Builder(this);
        tagbuilder.setTitle("选择分类");
        tagbuilder.setIcon(R.mipmap.ic_launcher);

        tagbuilder.setSingleChoiceItems(items, checkItem, new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                checkItem=which;
                button.setText(items[checkItem]);

            }
        });
        tagbuilder.setPositiveButton("确定", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                dialog.dismiss();
            }
        });
        alertDialog=tagbuilder.create();
        alertDialog.show();
    }

```

​     (3)修改NoteEditor.java中额UpdateNote方法将标签字段加入到数据库中

```java
    private final void updateNote(String text, String title) {
        Date nowTime = new Date(System.currentTimeMillis());
        SimpleDateFormat sdFormatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String retStrFormatNowDate = sdFormatter.format(nowTime);
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, retStrFormatNowDate);
        //标签字段
        values.put(NotePad.Notes.COLUMN_TAG_SELECTION_INDEX,items[checkItem]);
        values.put(NotePad.Notes.COLUMN_BACKGROUND_COLOR,colorBack);
        values.put(NotePad.Notes.COLUMN_TEXT_COLOR,colorText);
        values.put(NotePad.Notes.COLUMN_TEXT_NOTIFICATION_DATE,date+time);
        if (mState == STATE_INSERT) {
            if (title == null) {
                int length = text.length();
                title = text.substring(0, Math.min(5, length));
                if (length > 30) {
                    int lastSpace = title.lastIndexOf(' ');
                    if (lastSpace > 0) {
                        title = title.substring(0, lastSpace);
                    }
                }
            }
            values.put(NotePad.Notes.COLUMN_NAME_TITLE, title);

        } else if (title != null) {
            values.put(NotePad.Notes.COLUMN_NAME_TITLE, title);
        }
        values.put(NotePad.Notes.COLUMN_NAME_NOTE, text);
        getContentResolver().update(
                mUri,    // The URI for the record to update.
                values,  // The map of column names and new values to apply to them.
                null,    // No selection criteria are used, so no where columns are necessary.
                null     // No where columns are used, so no where arguments are necessary.
        );

    }
```

​    效果截图：

![image-20201220144924929](https://github.com/wangwang01-wzx/midterm/blob/main/image-20201220144924929.png)

![image-20201220144941990](https://github.com/wangwang01-wzx/midterm/blob/main/image-20201220144941990.png)

（4）显示侧滑菜单栏的效果

​       listview.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/dl"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:focusableInTouchMode="true"
        android:focusable="true"
       >
        <android.support.v7.widget.Toolbar
            android:id="@+id/tb"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="@color/colorPrimary"
            app:titleTextColor="@color/white"
            app:title="NotePad"
            app:navigationIcon="?attr/homeAsUpIndicator"
            >
        </android.support.v7.widget.Toolbar>
        <android.support.v7.widget.SearchView
            android:id="@+id/sv"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            >
        </android.support.v7.widget.SearchView>
        <android.support.design.widget.CoordinatorLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content">
            <ListView
                android:id="@+id/tv"
                android:layout_width="match_parent"
                android:layout_height="527dp"/>

        </android.support.design.widget.CoordinatorLayout>
    </LinearLayout>
    <android.support.design.widget.NavigationView
        android:id="@+id/nv"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:fitsSystemWindows="true"
        android:layout_gravity="left"
        app:headerLayout="@layout/nv_header"
        app:menu="@menu/nv_menu"
        >
    </android.support.design.widget.NavigationView>
</android.support.v4.widget.DrawerLayout>
```

   (5) NavigationView的菜单编写

​      nv_menu.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <group>
        <item android:title="@string/note">
            <menu>
                <item
                    android:id="@+id/notepad"
                    android:title="@string/note_name">
                </item>
            </menu>
        </item>
    </group>
    <group>
        <item android:title="@string/types">
            <menu>
                <item
                    android:id="@+id/travel"
                    android:title="@string/travel">
                </item>
                <item
                    android:id="@+id/work"
                    android:title="@string/work">
                </item>
                <item
                    android:id="@+id/study"
                    android:title="@string/study">
                </item>
                <item
                    android:id="@+id/life"
                    android:title="@string/life">
                </item>
                <item
                    android:id="@+id/def"
                    android:title="@string/other">
                </item>
            </menu>
        </item>
    </group>
</menu>
```

实现效果：

![image-20201220145447687](https://github.com/wangwang01-wzx/midterm/blob/main/image-20201220145447687.png)

​     （6)在NoteList.java中增加点击分类事件

​                 修改oncreate方法

```java
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.listview);
        // The user does not need to hold down the key to use menu shortcuts.
        setDefaultKeyMode(DEFAULT_KEYS_SHORTCUT);
        listView=findViewById(R.id.tv);
        toolbar=findViewById(R.id.tb);
        drawerLayout=findViewById(R.id.dl);
        setSupportActionBar(toolbar);
        toolbar.setNavigationOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                drawerLayout.openDrawer(Gravity.START);
            }
        });
        //navigation view settings
        navigationView=findViewById(R.id.nv);
        navigationView.setNavigationItemSelectedListener(new NavigationView.OnNavigationItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(@NonNull MenuItem menuItem) {
                switch (menuItem.getItemId()){
                    case R.id.notepad:
                        tagCursor = getContentResolver().query(
                                getIntent().getData(),            // Use the default content URI for the provider.
                                PROJECTION,                       // Return the note ID and title for each note.
                                null,                             // No where clause, return all records.
                                null,                             // No where clause, therefore no where column values.
                                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
                        );
                        adapter.swapCursor(tagCursor);
                        toolbar.setTitle("我的记事本");
                        drawerLayout.closeDrawer(navigationView);
                        break;
                    case R.id.travel:
                        tagSelection=NotePad.Notes.COLUMN_TAG_SELECTION_INDEX+" = '旅游'";
                        tagCursor = getContentResolver().query(
                                getIntent().getData(),            // Use the default content URI for the provider.
                                PROJECTION,                       // Return the note ID and title for each note.
                                tagSelection,                             // No where clause, return all records.
                                null,                             // No where clause, therefore no where column values.
                                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
                        );
                        adapter.swapCursor(tagCursor);
                        toolbar.setTitle("旅游");
                        drawerLayout.closeDrawer(navigationView);
                        break;
                    case R.id.work:
                        tagSelection=NotePad.Notes.COLUMN_TAG_SELECTION_INDEX+" = '工作'";
                        tagCursor = getContentResolver().query(
                                getIntent().getData(),            // Use the default content URI for the provider.
                                PROJECTION,                       // Return the note ID and title for each note.
                                tagSelection,                             // No where clause, return all records.
                                null,                             // No where clause, therefore no where column values.
                                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
                        );
                        adapter.swapCursor(tagCursor);
                        toolbar.setTitle("工作");
                        drawerLayout.closeDrawer(navigationView);
                        break;
                    case R.id.study:
                        tagSelection=NotePad.Notes.COLUMN_TAG_SELECTION_INDEX+" = '学习'";
                        tagCursor = getContentResolver().query(
                                getIntent().getData(),            // Use the default content URI for the provider.
                                PROJECTION,                       // Return the note ID and title for each note.
                                tagSelection,                             // No where clause, return all records.
                                null,                             // No where clause, therefore no where column values.
                                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
                        );
                        adapter.swapCursor(tagCursor);
                        toolbar.setTitle("学习");
                        drawerLayout.closeDrawer(navigationView);
                        break;
                    case R.id.life:
                        tagSelection=NotePad.Notes.COLUMN_TAG_SELECTION_INDEX+" = '生活'";
                        tagCursor = getContentResolver().query(
                                getIntent().getData(),            // Use the default content URI for the provider.
                                PROJECTION,                       // Return the note ID and title for each note.
                                tagSelection,                             // No where clause, return all records.
                                null,                             // No where clause, therefore no where column values.
                                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
                        );
                        adapter.swapCursor(tagCursor);
                        toolbar.setTitle("生活");
                        drawerLayout.closeDrawer(navigationView);
                        break;
                    case R.id.def:
                        tagSelection=NotePad.Notes.COLUMN_TAG_SELECTION_INDEX+" = '其它'";
                        tagCursor = getContentResolver().query(
                                getIntent().getData(),            // Use the default content URI for the provider.
                                PROJECTION,                       // Return the note ID and title for each note.
                                tagSelection,                             // No where clause, return all records.
                                null,                             // No where clause, therefore no where column values.
                                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
                        );
                        adapter.swapCursor(tagCursor);
                        toolbar.setTitle("其它");
                        drawerLayout.closeDrawer(navigationView);
                        break;
                }
                return true;
            }
        });
        SearchView();
        searchView.clearFocus();
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);

        }
        listView.setOnCreateContextMenuListener(this);
        Cursor cursor = getContentResolver().query(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note.
                null,                             // No where clause, return all records.
                null,                             // No where clause, therefore no where column values.
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );


        adapter
            = new SimpleCursorAdapter(
                      this,                             // The Context for the ListView
                      R.layout.noteslist_item,          // Points to the XML for a list item
                      cursor,                           // The cursor to get items from
                      dataColumns,
                      viewIDs,
                      CursorAdapter.FLAG_REGISTER_CONTENT_OBSERVER
              );
        requestPower();
 

        listView.setAdapter(adapter);
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                // Constructs a new URI from the incoming URI and the row ID
                Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);

                // Gets the action from the incoming Intent
                String action = getIntent().getAction();

                // Handles requests for note data
                if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {

                    // Sets the result to return to the component that called this Activity. The
                    // result contains the new URI
                    setResult(RESULT_OK, new Intent().setData(uri));
                } else {

                    // Sends out an Intent to start an Activity that can handle ACTION_EDIT. The
                    // Intent's data is the note ID URI. The effect is to call NoteEdit.
                    startActivity(new Intent(Intent.ACTION_EDIT, uri));
                }
            }
        });

        LoaderManager loaderManager=getLoaderManager();
        loaderManager.initLoader(0,null, this);
    }
```

效果截图

![image-20201220150220653](https://github.com/wangwang01-wzx/midterm/blob/main/image-20201220150220653.png)

![image-20201220150241908](https://github.com/wangwang01-wzx/midterm/blob/main/image-20201220150241908.png)

四： 实现UI美化

   （1）首页导航栏

​            1.   在values中添加style.xml文件   

```xml
<resources>
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/green</item>
    </style>

</resources>
```

   2.    修改AndroidManifest.xml添加主题

      ```xml
        <application
              android:allowBackup="true"
              android:icon="@drawable/app_notes"
              android:label="@string/app_name"
              android:roundIcon="@mipmap/ic_launcher_round"
              android:supportsRtl="true"
              android:theme="@style/AppTheme">
              </application>
      ```

      3. 修改list_options_menu.xml 更换新增按钮

         ```xml
         <?xml version="1.0" encoding="utf-8"?>
         <menu xmlns:tools="http://schemas.android.com/tools"
             xmlns:android="http://schemas.android.com/apk/res/android"
             xmlns:app="http://schemas.android.com/apk/res-auto">
             <!--  This is our one standard application action (creating a new note). -->
             <item android:id="@+id/menu_add"
                   android:icon="@drawable/newwzx"
                   android:alphabeticShortcut='a'
                   app:showAsAction="always"
                 android:title="new note" />
             <!--  If there is currently data in the clipboard, this adds a PASTE menu item to the menu
                   so that the user can paste in the data.. -->
             <item android:id="@+id/menu_paste"
                   android:icon="@drawable/ic_menu_compose"
                   android:title="@string/menu_paste"
                   android:alphabeticShortcut='p'
                   />
             <item
                 android:id="@+id/media_route_menu_item"
                 android:title="Cast"
                 app:actionProviderClass="android.support.v7.app.MediaRouteActionProvider"
                 tools:icon="@drawable/mr_button_light" />
         </menu>
         ```

         效果截图：

         ![image-20201220151152861](https://github.com/wangwang01-wzx/midterm/blob/main/image-20201220151152861.png)

（2）修改背景颜色

​      1.在editor_options_menu.xml中添加修改颜色这一选项，并划分为两类，修改背景颜色和修改字体颜色

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:background="@color/colorPrimary">
    <item android:id="@+id/menu_save"
          android:icon="@drawable/ic_menu_save"
          android:alphabeticShortcut='s'
          android:title="@string/menu_save"
          app:showAsAction="always|withText" />
    <item android:id="@+id/menu_revert"
          android:icon="@drawable/ic_menu_revert"
          android:title="@string/menu_revert"
        app:showAsAction="always|withText"
        />
    <item android:id="@+id/menu_delete"
          android:icon="@drawable/ic_menu_delete"
          android:title="@string/menu_delete"
          app:showAsAction="always|withText" />
    <item
        android:title="改变颜色">
        <menu>
            <item
                android:title="改变背景颜色"
                android:id="@+id/background-color">
            </item>
           
        </menu>
    </item>

</menu>
```

效果截图

![image-20201220151923594](https://github.com/wangwang01-wzx/midterm/blob/main/image-20201220151923594.png)

![image-20201220153021169](https://github.com/wangwang01-wzx/midterm/blob/main/image-20201220153021169.png)

2.采用AlertDialog来实现界面交互效果，修改颜色调用的是函数showColor()，在NoteEditor增加点击事件

```java
  private void showColor(){
        AlertDialog alertDialog=new AlertDialog.Builder(this).setTitle("请选择颜色").
               setView(R.layout.color_layout)
                .setPositiveButton("确定", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        dialog.dismiss();
                    }
                }).create();
        alertDialog.show();
    }
```

点击事件

```java
   public void onClick(View v) {
        switch (v.getId()){
            case R.id.orange:
                if(isFlag){
                    mText.setBackgroundColor(Color.parseColor("#FF8C00"));
                    colorBack="#FF8C00";
                }else{
                    mText.setTextColor(Color.parseColor("#FF8C00"));
                    colorText="#FF8C00";
                }
                break;

            case R.id.aqua:
                if(isFlag){
                    mText.setBackgroundColor(Color.parseColor("#00FFFF"));
                    colorBack="#00FFFF";
                }else{
                    mText.setTextColor(Color.parseColor("#00FFFF"));
                    colorText="#00FFFF";
                }
                break;

            case R.id.pink:
                if(isFlag){
                    mText.setBackgroundColor(Color.parseColor("#D81B60"));
                    colorBack="#D81B60";
                }else{
                    mText.setTextColor(Color.parseColor("#D81B60"));
                    colorText="#D81B60";
                }
                break;
            case R.id.green:
                if(isFlag){
                    mText.setBackgroundColor(Color.parseColor("#00FF7F"));
                    colorBack="#00FF7F";
                }else{
                    mText.setTextColor(Color.parseColor("#00FF7F"));
                    colorText="#00FF7F";
                }
                break;
        }
    }

```



3.AlertDialog布局文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">
    <Button
        android:id="@+id/orange"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:background="@color/orange"
        android:layout_weight="1"
        android:onClick="onClick"/>
    <Button
        android:id="@+id/aqua"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:background="@color/aqua"
        android:layout_weight="1"
        android:onClick="onClick"/>

    <Button
        android:id="@+id/pink"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:background="@color/colorAccent"
        android:layout_weight="1"
        android:onClick="onClick"/>
    <Button
        android:id="@+id/green"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:background="@color/green"
        android:layout_weight="1"
        android:onClick="onClick"/>
</LinearLayout>

```

效果截图：

![image-20201220153436346](https://github.com/wangwang01-wzx/midterm/blob/main/image-20201220153436346.png)

![image-20201220153454636](https://github.com/wangwang01-wzx/midterm/blob/main/image-20201220153454636.png)

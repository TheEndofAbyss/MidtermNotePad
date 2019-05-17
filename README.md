# MidtermNotePad
这是期中实验-记事本
**期中时间-记事本**
一、基本功能

（1）搜索
1.在listview.xml中添加SearchView控件：

```
<android.support.v7.widget.SearchView
    android:id="@+id/sv"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    >
 ```
 
2.在NodeList.java中创建一个SeachView函数来配置SeachView：

```
private void SearchView(){
    searchView=findViewById(R.id.sv);
    searchView.onActionViewExpanded();
    searchView.setQueryHint("搜索笔记");
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
                        NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
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

3.结果截图：

（2）时间戳

1.在notelist_item中添加一个TextView用来显示时间戳：

```
<TextView
    android:id="@+id/text2"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:textAppearance="?android:attr/textAppearanceLarge"
    android:textSize="12dp"
    android:gravity="center_vertical"
    android:paddingLeft="10dip"
    android:singleLine="true"
    android:layout_weight="1"
    android:layout_margin="0dp"
    />
 ```
 
 2.在NoteEditor.java中已有创建和修改时间的两个字段：
 
```
Date nowTime = new Date(System.currentTimeMillis());
SimpleDateFormat sdFormatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
String retStrFormatNowDate = sdFormatter.format(nowTime);
```

3.在NodeEditor.java中的updateNode()函数函数中修改字段，用以将时间存入数据库：

```
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, retStrFormatNowDate);
```
4.在NoteList.java的PROJECTION数组中增加该字段的描述：

```
private static final String[] PROJECTION =
    new String[] {
        NotePad.Notes._ID,
        NotePad.Notes.COLUMN_NAME_TITLE,
        NotePad.Notes.COLUMN_NAME_NOTE
};
```

5.在SimpleCursorAdapter中的参数viewsIDs和dataColumns增加子段描述(NotesList.java):

```
private int[] viewIDs = { R.id.text,R.id.text2};
```

```
private String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;
```

```
adapter
    = new SimpleCursorAdapter(
              this,                             // The Context for the ListView
              R.layout.noteslist_item,          // Points to the XML for a list item
              cursor,                           // The cursor to get items from
              dataColumns,
              viewIDs,
              CursorAdapter.FLAG_REGISTER_CONTENT_OBSERVER
      );
   ```

6.结果截图：
二、额外功能

（1）导出文件

1.在editor_options_menu.xml中添加一个导出笔记的选项：

```
<item android:id="@+id/menu_output"
    android:title="@string/menu_output" />
```

2.在onOptionsItemSelected()方法菜单的switch中添加(NoteEditor.java)：

```
  case R.id.menu_output:
        outputNote();
        break;
  ```

3.在NoteEditor.java中添加函数outputNote()：

```
    private final void outputNote() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,OutputText.class);
        NoteEditor.this.startActivity(intent);
    }
 ```

4.新建布局output_text.xml，垂直线性布局放置EditText和Button：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">
    <EditText android:id="@+id/output_name"
        android:maxLines="1"
        android:layout_marginTop="2dp"
        android:layout_marginBottom="15dp"
        android:layout_width="wrap_content"
        android:ems="25"
        android:layout_height="wrap_content"
        android:autoText="true"
        android:capitalize="sentences"
        android:scrollHorizontally="true" />
    <Button android:id="@+id/output_ok"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:text="@string/output_ok"
        android:onClick="OutputOk" />
</LinearLayout>
```

5.新建OutputText.java用来选择颜色：

```
public class OutputText extends Activity {
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_NOTE, // 2
            NotePad.Notes.COLUMN_NAME_CREATE_DATE, // 3
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 4
    };
    private String TITLE;
    private String NOTE;
    private String CREATE_DATE;
    private String MODIFICATION_DATE;
    private Cursor mCursor;
    private EditText mName;
    private Uri mUri;
    private boolean flag = false;
    private static final int COLUMN_INDEX_TITLE = 1;
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.output_text);
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,    
                PROJECTION,  
                null,    
                null,     
                null      
        );
        mName = (EditText) findViewById(R.id.output_name);
    }
    @Override
    protected void onResume(){
        super.onResume();
        if (mCursor != null) {
            mCursor.moveToFirst();
            mName.setText(mCursor.getString(COLUMN_INDEX_TITLE));
        }
    }
    @Override
    protected void onPause() {
        super.onPause();
        if (mCursor != null) {
            TITLE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE));
            NOTE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE));
            CREATE_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_CREATE_DATE));
            MODIFICATION_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE));
            if (flag == true) {
                write();
            }
            flag = false;
        }
    }
    public void OutputOk(View v){
        flag = true;
        finish();
    }
    private void write()
    {
        try
        {
            if (Environment.getExternalStorageState().equals(
                    Environment.MEDIA_MOUNTED)) {
                File sdCardDir = Environment.getExternalStorageDirectory();
                File targetFile = new File(sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt");
                PrintWriter ps = new PrintWriter(new OutputStreamWriter(new FileOutputStream(targetFile), "UTF-8"));
                ps.println(TITLE);
                ps.println(NOTE);
                ps.println("创建时间：" + CREATE_DATE);
                ps.println("最后一次修改时间：" + MODIFICATION_DATE);
                ps.close();
                Toast.makeText(this, "保存成功,保存位置：" + sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt", Toast.LENGTH_LONG).show();
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```

6.在AndroidManifest.xml中将这个Acitvity主题定义为对话框样式，并且加入权限：

```
       <activity android:name="OutputText"
            android:label="@string/output_name"
            android:theme="@android:style/Theme.Holo.Dialog"
            android:windowSoftInputMode="stateVisible">
        </activity>
 ```

```
<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

7.结果截图：


（2）在文件中添加图片

1.在editor_options_menu.xml中添加添加选项：

```
<item android:title="插入图片"
    android:icon="@android:drawable/ic_menu_gallery"
    app:showAsAction="always|withText">
    <menu>
        <item
            android:title="相册"
            android:icon="@android:drawable/ic_menu_gallery"
            android:id="@+id/insert_album">
        </item>
        <item
            android:id="@+id/insert_camera"
            android:icon="@android:drawable/ic_menu_camera"
            android:title="拍照">
        </item>
    </menu>
</item>
```

2.在NoteEditor.java中设置图片获取方式：

```
//相册
public void getPhoto() {
    Intent intent = new Intent();
    intent.setType("image/*");
    intent.setAction(Intent.ACTION_OPEN_DOCUMENT);
    startActivityForResult(intent, PHOTO_FROM_GALLERY);
}
//拍照
public void takeCamera() {
    if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
        ActivityCompat.requestPermissions( this, new String[] { Manifest.permission.CAMERA }, PHOTO_FROM_CAMERA);
    }
    else {
        File file=new File(Environment.getExternalStorageDirectory(),System.currentTimeMillis()+".jpg");
        try {
            if(file.exists()){
                file.delete();
            }
            file.createNewFile();
        }catch (IOException e){
            e.printStackTrace();
        }
        imageUri=Uri.fromFile(file);
        Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        intent.putExtra(MediaStore.EXTRA_OUTPUT,imageUri);
        startActivityForResult(intent, PHOTO_FROM_CAMERA);
    }
}
```

3.在AndroidManifest.xml中设置权限：

```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

4.在NoteEditor.java中编写onActivityResult定义返回操作：

```
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    ContentResolver resolver = getContentResolver();
    super.onActivityResult(requestCode, resultCode, data);
    switch (requestCode) {
        case PHOTO_FROM_GALLERY:
            switch (resultCode) {
                case RESULT_OK:
                    if (data != null) {
                        Uri uri = data.getData();
                        Bitmap bitmap = null;
                        try {
                            Bitmap originalBitmap = BitmapFactory.decodeStream(resolver.openInputStream(uri));
                            bitmap = resizeImage(originalBitmap, 200, 200);
                        } catch (FileNotFoundException e) {
                            e.printStackTrace();
                        }
                        if(bitmap != null){
                            int index = mText.getSelectionStart();
                            Editable edit_text = mText.getEditableText();
                            if(index <0 || index >= edit_text.length()){
                                edit_text.append(uri.toString());
                                updateNoteText(mText.getText().toString());
                            }else{
                                edit_text.insert(index,uri.toString());
                                updateNoteText(mText.getText().toString());
                            }
                        }else{
                            Toast.makeText(NoteEditor.this, "获取图片失败", Toast.LENGTH_SHORT).show();
                        }
                    }
                    break;
                case RESULT_CANCELED:
                    break;
            }
            break;
        case PHOTO_FROM_CAMERA:
            if (resultCode == RESULT_OK) {
                Bitmap originalBitmap1=null;
                try{
                    originalBitmap1=BitmapFactory.decodeStream(getContentResolver().openInputStream(imageUri));
                }catch (FileNotFoundException e){
                    e.printStackTrace();
                }
                if(originalBitmap1 != null){
                    int index = mText.getSelectionStart(); 
                    Editable edit_text = mText.getEditableText();
                    if(index <0 || index >= edit_text.length()){
                        edit_text.append(imageUri.toString());
                        updateNoteText(mText.getText().toString());
                    }else{
                        edit_text.insert(index, imageUri.toString());
                        updateNoteText(mText.getText().toString());
                    }
                }else{
                    Toast.makeText(NoteEditor.this, "获取图片失败", Toast.LENGTH_SHORT).show();
                }

            } else {
                Log.e("result", "is not ok" + resultCode);
            }
            break;
        default:
            break;
    }
}
```

5.在NoteEditor.java中定义两个正则表达式，提取文本的图片路径，实现图片插入：

```
private static final String regex="content://com.android.providers.media.documents/"
        +"document/image%\\w{4}";
private static final String reg="file:///storage/emulated/0/\\d+.jpg";
```

6.在NoteEditor.java的onResumn（）和cancelNote（）中添加代码：

```
int colNoteIndex = mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE);

String note = mCursor.getString(colNoteIndex);
ArrayList<String> contentList=new ArrayList<>();
ArrayList<Integer> startList=new ArrayList<>();
ArrayList<Integer> endList=new ArrayList<>();
Pattern p=Pattern.compile(regex);
Matcher m=p.matcher(note);

while(m.find()){
    contentList.add(m.group());
    startList.add(m.start());
    endList.add(m.end());
    flag=true;
}
p=Pattern.compile(reg);
m=p.matcher(note);
while(m.find()){
    contentList.add(m.group());
    startList.add(m.start());
    endList.add(m.end());
    flag=true;
}

if(!flag){
    mText.setText(note);
}else{
    pushPicture(note,contentList,startList,endList);
}
```

7.在NoteEditor.java的pushPicture（）中对图片进行处理：

```
private void pushPicture(String note,ArrayList<String> contentList,ArrayList<Integer> startList,ArrayList<Integer> endList) {
    SpannableString spannableString = new SpannableString(note);
    for(int i=0;i<contentList.size();i++) {
        Uri uri = Uri.parse(contentList.get(i));
        Bitmap bitmap = null;
        try {
            Bitmap originalBitmap = BitmapFactory.decodeStream(getContentResolver().openInputStream(uri));
            bitmap = resizeImage(originalBitmap, 200, 200);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        if (bitmap != null) {
            ImageSpan imageSpan = new ImageSpan(NoteEditor.this, bitmap);
            spannableString.setSpan(imageSpan, startList.get(i), endList.get(i), Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
        }
    }
    mText.setText("");
    Editable edit_text = mText.getEditableText();
    edit_text.append(spannableString);
}
```

8.结果截图：

（3）更改字体大小及颜色

1.在editor_options_menu.xml中添加字体颜色大小选项：

```
<item android:title="字体大小">
    <menu>
        <group>
            <item android:id="@+id/font_10"
                android:title="小"/>
            <item android:id="@+id/font_16"
                android:title="中"/>
            <item android:id="@+id/font_20"
                android:title="大"/>
        </group>
    </menu>
</item>
<item
    android:title="字体颜色">
    <menu>
        <group>
            <item android:id="@+id/red_font"
                android:title="红色"/>
            <item android:id="@+id/black_font"
                android:title="黑色"/>
        </group>
    </menu>
</item>
```

2.在NoteEditor.java的onOptionsItemSelected（）中添加switch选项：

```
case R.id.font_10:
    mText.setTextSize(20);
        break;
    case R.id.font_16:
        mText.setTextSize(32);
        break;
    case R.id.font_20:
        mText.setTextSize(40);
        break;
    case R.id.red_font:
        mText.setTextColor(Color.RED);
        break;
    case R.id.black_font:
        mText.setTextColor(Color.BLACK);
        break;
 ```

3.结果截图：


（4）添加背景图片

1.在想添加背景图片的布局文件中加入代码（LinearLayout中）：

```
android:background="@drawable/你的背景图片名"
```

2.结果截图：

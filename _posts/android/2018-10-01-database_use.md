---
layout: post
title: '[Android] Android sqlight database usage(안드로이드 sqlight 데이터베이스 사용)'
tags: [android]
description: >
    Android sqlight database usage(안드로이드 sqlight 데이터베이스 사용)
---

데이터베이스(Databse, db)는 체계화된 데이터의 모임이다. 논리적으로 연관된 하나 이상의 자료의 모음으로 그 내용을 고도로 구조화함으로써 검색과 갱신의 효율화를 꾀한 것이다. 즉, 몇 개의 자료 파일을 조직적으로 통합하여 자료 항목의 중복을 없애고 자료를 구조화하여 기억시켜 놓은 자료의 집합체라고 할 수 있다. 자세한 데이터베이스에 대한 내용은 [databse wiki](https://ko.wikipedia.org/wiki/데이터베이스)를 참고하자.

안드로이드에서도 데이터를 관리하기 위해서 db를 사용하면 좋다. 이 글에서는 안드로이드에서 사용하기 편리한 sqlight를 사용하여 데이터를 관리하는 방법에 대해서 소개한다. 

## 1. DB 관리 클레스 생성 (DBHelper)

Activity에서 DB를 잘 관리하기 위해서 DB를 관리하기 위한 DBHelper라는 클래스를 생성한다. 아래 예제코드에서는 1개의 앱에서는 1개의 db파일만을 관리하기 때문에 생성자에서 db파일을 생성해서 관리한다. 만약에 사용자별로 db를 관리하거나 생성하고자 한다면 생성자에서 db파일을 생성하지 않고 추가 생성관련 함수에서 db를 생성하면 된다.

```
import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.DatabaseUtils;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
import android.os.Environment;
import android.util.Log;
import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;
import java.io.File;
import java.time.LocalDateTime;

public class DBHelper extends SQLiteOpenHelper {

    Context context;
    MainActivity activity;
    String tb_name;
    int version;
    SQLiteDatabase.CursorFactory factory;

    // DBHelper 생성자로 관리할 DB 이름과 버전 정보를 받음
    public DBHelper(Context context, String name, SQLiteDatabase.CursorFactory factory, int version) {
        super(context, name, factory, version);
        this.context = context;
        tb_name = name;
        this.version = version;
        this.factory = factory;
        SQLiteDatabase db = getWritableDatabase();
        db.execSQL("CREATE TABLE IF NOT EXISTS " + name + " (_id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                "time DATETIME DEFAULT (datetime('now','localtime')), patient_name VARCHAR2(100), patient_age INT, patient_gender INT, " +
                "patient_number INT, patient_time INT, ave_speed DOUBLE, physiological_age DOUBLE, fraility_index DOUBLE," +
                "raw_file_path VARCHAR2(100));");
    }

    @Override
    public void onCreate(SQLiteDatabase db) {

    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }

	//데이터 베이스 데이터의 갯수를 출력
    public long getDataSize() {
        SQLiteDatabase db = this.getReadableDatabase();
        long count = DatabaseUtils.queryNumEntries(db, tb_name);
        db.close();
        return count;
    }

	//새로운 데이터 입력
    public void insert(String patient_name, int patient_age, int patient_gender, int patient_number, int patient_time, double ave_speed, double physiological_age, double fraility_index, String raw_file_path) {
        // 읽고 쓰기가 가능하게 DB 열기
        SQLiteDatabase db = getWritableDatabase();
        // DB에 입력한 값으로 행 추가
        ContentValues values = new ContentValues();
        values.put("patient_name", patient_name);
        values.put("patient_age", patient_age);
        values.put("patient_gender", patient_gender);
        values.put("patient_number", patient_number);
        values.put("patient_time", patient_time);
        values.put("ave_speed", ave_speed);
        values.put("physiological_age", physiological_age);
        values.put("fraility_index", fraility_index);
        values.put("raw_file_path", raw_file_path);
        db.insert(tb_name, null, values);

        db.close();
    }

	//기존에 존재하는 데이터 업데이트
    public void update(int id, String patient_name, int patient_age, int patient_gender, int patient_number, int patient_time, double ave_speed, double physiological_age, double fraility_index, String raw_file_path) {
        // 읽고 쓰기가 가능하게 DB 열기
        SQLiteDatabase db = getWritableDatabase();
        // DB에 입력한 값으로 행 추가
        ContentValues values = new ContentValues();
        values.put("patient_name", patient_name);
        values.put("patient_age", patient_age);
        values.put("patient_gender", patient_gender);
        values.put("patient_number", patient_number);
        values.put("patient_time", patient_time);
        values.put("ave_speed", ave_speed);
        values.put("physiological_age", physiological_age);
        values.put("fraility_index", fraility_index);
        values.put("raw_file_path", raw_file_path);
        db.update(tb_name, values, "_id=?", new String[]{String.valueOf(id)});
        db.close();
    }
	
	//특정 id를 갖는 데이터를 json으로 출력
    public JSONObject getData(int id) {
        Log.i("ID", String.valueOf(id));
        SQLiteDatabase db = getReadableDatabase();
        String query = "SELECT * FROM " + tb_name + " WHERE _id = ?";
        Cursor cursor = db.rawQuery(query, new String[]{String.valueOf(id)});
        cursor.moveToPosition(0);

        int totalColumn = cursor.getColumnCount();
        JSONObject rowObject = new JSONObject();

        for (int i = 0; i < totalColumn; i++) {
            if (cursor.getColumnName(i) != null) {
                try {
                    if (cursor.getString(i) != null) {
                        rowObject.put(cursor.getColumnName(i), cursor.getString(i));
                    } else {
                        rowObject.put(cursor.getColumnName(i), "");
                    }
                } catch (Exception e) {
                
                }
            }
        }
        cursor.close();
        db.close();
        return rowObject;
    }

	//시작 id부터 일정 갯수의 데이터를 json array로 얻어온다
    public JSONArray getResult(int start_index, int number_of_data) {
        // 읽기가 가능하게 DB 열기
        SQLiteDatabase db = getReadableDatabase();
        String result = "";

        // DB에 있는 데이터를 쉽게 처리하기 위해 Cursor를 사용하여 테이블에 있는 모든 데이터 출력
        Cursor cursor = db.rawQuery("SELECT * FROM " + tb_name, null);
        JSONArray resultSet = new JSONArray();
        cursor.moveToPosition(start_index);
        int index = 0;
        while (index < number_of_data) {

            int totalColumn = cursor.getColumnCount();
            JSONObject rowObject = new JSONObject();

            for (int i = 0; i < totalColumn; i++) {
                if (cursor.getColumnName(i) != null) {
                    try {
                        if (cursor.getString(i) != null) {
                            rowObject.put(cursor.getColumnName(i), cursor.getString(i));
                        } else {
                            rowObject.put(cursor.getColumnName(i), "");
                        }
                    } catch (Exception e) {

                    }
                }

            }
            resultSet.put(rowObject);
            cursor.moveToNext();
            index = index + 1;
        }

        cursor.close();
        return resultSet;
    }

	//최근 데이터로 부터 일정 갯수의 데이터를 json array 로 얻어온다.
    public JSONArray getRecentData(int number_of_data) {
        // 읽기가 가능하게 DB 열기
        SQLiteDatabase db = getReadableDatabase();
        String result = "";

        // DB에 있는 데이터를 쉽게 처리하기 위해 Cursor를 사용하여 테이블에 있는 모든 데이터 출력
        Cursor cursor = db.rawQuery("SELECT * FROM " + tb_name, null);
        JSONArray resultSet = new JSONArray();
        cursor.moveToLast();
        int index = 0;
        while (index < number_of_data) {

            int totalColumn = cursor.getColumnCount();
            JSONObject rowObject = new JSONObject();

            for (int i = 0; i < totalColumn; i++) {
                if (cursor.getColumnName(i) != null) {
                    try {
                        if (cursor.getString(i) != null) {
                            rowObject.put(cursor.getColumnName(i), cursor.getString(i));
                        } else {
                            rowObject.put(cursor.getColumnName(i), "");
                        }
                    } catch (Exception e) {

                    }
                }

            }
            resultSet.put(rowObject);
            cursor.moveToPrevious();
            index = index + 1;
        }

        cursor.close();
        return resultSet;
    }

	//Primary key인 id를 이용해서 검색 후 삭제
    public void delete(int item) throws JSONException {

        // 입력한 항목과 일치하는 행 삭제
        SQLiteDatabase db = getWritableDatabase();
        db.execSQL("DELETE FROM " + tb_name + " WHERE _id= " + item + ";");
        db.close();
    }
    
	//모든 데이터 삭제
    public void removeAll() {
        SQLiteDatabase db = getReadableDatabase(); // helper is object extends SQLiteOpenHelper
        db.execSQL("DROP TABLE " + tb_name);
        db.execSQL("CREATE TABLE IF NOT EXISTS " + tb_name + " (_id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                "time DATETIME DEFAULT (datetime('now','localtime')), patient_name VARCHAR2(100), patient_age INT, patient_gender INT, " +
                "patient_number INT, patient_time INT, ave_speed DOUBLE, physiological_age DOUBLE, fraility_index DOUBLE," +
                "raw_file_path VARCHAR2(100));");

    }

}
```
중요한 부분만 발췌해서 설명하면 다음과 같다. 


DBHelper 클래스의 생성자이다. 생성자의 입력으로 들어온 table의 이름으로 db table을 생성하며, db table이 없을때에만 새로 생성한다. primary key로는 id가 되며 데이터가 입력될 때마다 1씩 자동으로 증가한다.

```
public DBHelper(Context context, String name, SQLiteDatabase.CursorFactory factory, int version) {
	super(context, name, factory, version);
	this.context = context;
	tb_name = name;
	this.version = version;
	this.factory = factory;
	SQLiteDatabase db = getWritableDatabase();
	db.execSQL("CREATE TABLE IF NOT EXISTS " + name + " (_id INTEGER PRIMARY KEY AUTOINCREMENT, " +
			"time DATETIME DEFAULT (datetime('now','localtime')), patient_name VARCHAR2(100), patient_age INT, patient_gender INT, " +
			"patient_number INT, patient_time INT, ave_speed DOUBLE, physiological_age DOUBLE, fraility_index DOUBLE," +
			"raw_file_path VARCHAR2(100));");
}
```

데이터를 추가하는 함수이다. ContentValues에 입력된 데이터를 추가 후 insert 해준다.

```
public void insert(String patient_name, int patient_age, int patient_gender, int patient_number, int patient_time, double ave_speed, double physiological_age, double fraility_index, String raw_file_path) {
	// 읽고 쓰기가 가능하게 DB 열기
	SQLiteDatabase db = getWritableDatabase();
	// DB에 입력한 값으로 행 추가
	ContentValues values = new ContentValues();
	values.put("patient_name", patient_name);
	values.put("patient_age", patient_age);
	values.put("patient_gender", patient_gender);
	values.put("patient_number", patient_number);
	values.put("patient_time", patient_time);
	values.put("ave_speed", ave_speed);
	values.put("physiological_age", physiological_age);
	values.put("fraility_index", fraility_index);
	values.put("raw_file_path", raw_file_path);
	db.insert(tb_name, null, values);
	db.close();
}
```

기존에 있던 데이터를 변경해 준다. 이때는 어떤 데이터를 수정할 지 id 값을 입력해 줘야 한다.

```
public void update(int id, String patient_name, int patient_age, int patient_gender, int patient_number, int patient_time, double ave_speed, double physiological_age, double fraility_index, String raw_file_path) {
	// 읽고 쓰기가 가능하게 DB 열기
	SQLiteDatabase db = getWritableDatabase();
	// DB에 입력한 값으로 행 추가
	ContentValues values = new ContentValues();
	values.put("patient_name", patient_name);
	values.put("patient_age", patient_age);
	values.put("patient_gender", patient_gender);
	values.put("patient_number", patient_number);
	values.put("patient_time", patient_time);
	values.put("ave_speed", ave_speed);
	values.put("physiological_age", physiological_age);
	values.put("fraility_index", fraility_index);
	values.put("raw_file_path", raw_file_path);
	db.update(tb_name, values, "_id=?", new String[]{String.valueOf(id)});
	db.close();
}
```

특정 id의 데이터를 json파일로 출력한다. json파일은 구조화된 데이터를 갖는 객체이다.

```
//특정 id를 갖는 데이터를 json으로 출력
public JSONObject getData(int id) {
	SQLiteDatabase db = getReadableDatabase();
	String query = "SELECT * FROM " + tb_name + " WHERE _id = ?";
	Cursor cursor = db.rawQuery(query, new String[]{String.valueOf(id)});
	cursor.moveToPosition(0);

	int totalColumn = cursor.getColumnCount();
	JSONObject rowObject = new JSONObject();

	for (int i = 0; i < totalColumn; i++) {
		if (cursor.getColumnName(i) != null) {
			try {
				if (cursor.getString(i) != null) {
					rowObject.put(cursor.getColumnName(i), cursor.getString(i));
				} else {
					rowObject.put(cursor.getColumnName(i), "");
				}
			} catch (Exception e) {
			
			}
		}
	}
	cursor.close();
	db.close();
	return rowObject;
}
```

시작 id에서 부터 일정한 갯수의 데이터를 얻어오는 함수이다.

```
//시작 id부터 일정 갯수의 데이터를 json array로 얻어온다
public JSONArray getResult(int start_index, int number_of_data) {
	// 읽기가 가능하게 DB 열기
	SQLiteDatabase db = getReadableDatabase();
	String result = "";

	Cursor cursor = db.rawQuery("SELECT * FROM " + tb_name, null);
	JSONArray resultSet = new JSONArray();
	cursor.moveToPosition(start_index);
	int index = 0;
	while (index < number_of_data) {

		int totalColumn = cursor.getColumnCount();
		JSONObject rowObject = new JSONObject();

		for (int i = 0; i < totalColumn; i++) {
			if (cursor.getColumnName(i) != null) {
				try {
					if (cursor.getString(i) != null) {
						rowObject.put(cursor.getColumnName(i), cursor.getString(i));
					} else {
						rowObject.put(cursor.getColumnName(i), "");
					}
				} catch (Exception e) {

				}
			}

		}
		resultSet.put(rowObject);
		cursor.moveToNext();
		index = index + 1;
	}
	cursor.close();
	return resultSet;
}
```

가장 최근 데이터 (가장 큰 id를 갖는 데이터)로 부터 일정 갯수의 데이터를 출력하는 함수

```
//최근 데이터로 부터 일정 갯수의 데이터를 json array 로 얻어온다.
public JSONArray getRecentData(int number_of_data) {
	// 읽기가 가능하게 DB 열기
	SQLiteDatabase db = getReadableDatabase();
	String result = "";

	// DB에 있는 데이터를 쉽게 처리하기 위해 Cursor를 사용하여 테이블에 있는 모든 데이터 출력
	Cursor cursor = db.rawQuery("SELECT * FROM " + tb_name, null);
	JSONArray resultSet = new JSONArray();
	cursor.moveToLast();
	int index = 0;
	while (index < number_of_data) {

		int totalColumn = cursor.getColumnCount();
		JSONObject rowObject = new JSONObject();

		for (int i = 0; i < totalColumn; i++) {
			if (cursor.getColumnName(i) != null) {
				try {
					if (cursor.getString(i) != null) {
						rowObject.put(cursor.getColumnName(i), cursor.getString(i));
					} else {
						rowObject.put(cursor.getColumnName(i), "");
					}
				} catch (Exception e) {

				}
			}

		}
		resultSet.put(rowObject);
		cursor.moveToPrevious();
		index = index + 1;
	}

	cursor.close();
	return resultSet;
}
```

특정 id를 갖는 데이터만 삭제, 이렇게 데이터가 삭제된 후 데이터가 새로 입력되어도 삭제된 데이터의 id는 채워지지 않고 계속 id값은 증가한다.

```
//Primary key인 id를 이용해서 검색 후 삭제
public void delete(int item) throws JSONException {

	// 입력한 항목과 일치하는 행 삭제
	SQLiteDatabase db = getWritableDatabase();
	db.execSQL("DELETE FROM " + tb_name + " WHERE _id= " + item + ";");
	db.close();
}
```

table에 있는 모든 데이터를 삭제 후 테이블 재 생성. 이런경우에 새로 데이터가 입력되면 id는 처음부터 새로 시작한다.

```
//모든 데이터 삭제, 
public void removeAll() {
	SQLiteDatabase db = getReadableDatabase(); // helper is object extends SQLiteOpenHelper
	db.execSQL("DROP TABLE " + tb_name);
	db.execSQL("CREATE TABLE IF NOT EXISTS " + tb_name + " (_id INTEGER PRIMARY KEY AUTOINCREMENT, " +
			"time DATETIME DEFAULT (datetime('now','localtime')), patient_name VARCHAR2(100), patient_age INT, patient_gender INT, " +
			"patient_number INT, patient_time INT, ave_speed DOUBLE, physiological_age DOUBLE, fraility_index DOUBLE," +
			"raw_file_path VARCHAR2(100));");

}
```

## DBHelper 사용

Mainactivity에 DBHelper 객체를 선언하고 생성한다.

```
DBHelper user_db;
user_db = new DBHelper(this, "user_db", null, 1);
```

객체를 생성한 후에 객채의 맴버 함수를 사용하여 입력, 검색, 삭제 등을 수행하면 된다.

다음은 db로부터 데이터를 json 객체로 받아오는 과정이다.

```
JSONObject data = user_db.getData(data_index);
try {
	time = data.getString("time");
	patient_name = data.getString("patient_name");
	patient_age = data.getInt("patient_age");
	patient_gender = data.getInt("patient_gender");
	patient_number = data.getInt("patient_number");
	patient_average_speed = data.getDouble("ave_speed");
	rawfile = data.getString("raw_file_path");
} catch (JSONException e) {
e.printStackTrace();
}
```



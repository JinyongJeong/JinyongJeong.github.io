---
layout: post
title: '[Android] Android sqlight database useage(안드로이드 sqlight 데이터베이스 사용)'
tags: [android]
description: >
    Android sqlight database useage(안드로이드 sqlight 데이터베이스 사용)
---

데이터베이스(Databse, db)는 체계화된 데이터의 모임이다. 논리적으로 연관된 하나 이상의 자료의 모음으로 그 내용을 고도로 구조화함으로써 검색과 갱신의 효율화를 꾀한 것이다. 즉, 몇 개의 자료 파일을 조직적으로 통합하여 자료 항목의 중복을 없애고 자료를 구조화하여 기억시켜 놓은 자료의 집합체라고 할 수 있다. 자세한 데이터베이스에 대한 내용은 [databse wiki](https://ko.wikipedia.org/wiki/데이터베이스)를 참고하자.

안드로이드에서도 데이터를 관리하기 위해서 db를 사용하면 좋다. 이 글에서는 안드로이드에서 사용하기 편리한 sqlight를 사용하여 데이터를 관리하는 방법에 대해서 소개한다. 

## 1. DB 관리 클레스 생성 (DBHelper)

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

    // DBHelper 생성자로 관리할 DB 이름과 버전 정보를 받음
    public DBHelper(Context context, String name, SQLiteDatabase.CursorFactory factory, int version) {
        super(context, name, factory, version);
        this.context = context;
        SQLiteDatabase db = getWritableDatabase();
        db.execSQL("CREATE TABLE IF NOT EXISTS " + "tb_data" + " (_id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                "time DATETIME DEFAULT (datetime('now','localtime')), patient_name VARCHAR2(100), patient_age INT, patient_gender INT, " +
                "patient_number INT, patient_time INT, ave_speed DOUBLE, physiological_age DOUBLE, fraility_index DOUBLE," +
                "raw_file_path VARCHAR2(100));");

    }

    @Override
    public void onCreate(SQLiteDatabase db) {

        // 새로운 테이블 생성
        /* 이름은 MONEYBOOK이고, 자동으로 값이 증가하는 _id 정수형 기본키 컬럼과
        item 문자열 컬럼, price 정수형 컬럼, create_at 문자열 컬럼으로 구성된 테이블을 생성. */

        db.execSQL("CREATE TABLE IF NOT EXISTS " + "tb_data" + " (_id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                "time DATETIME DEFAULT (datetime('now','localtime')), patient_name VARCHAR2(100), patient_age INT, patient_gender INT, " +
                "patient_number INT, patient_time INT, ave_speed DOUBLE, physiological_age DOUBLE, fraility_index DOUBLE," +
                "raw_file_path VARCHAR2(100));");

    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }

    public long getDataSize() {
        SQLiteDatabase db = this.getReadableDatabase();
        long count = DatabaseUtils.queryNumEntries(db, "tb_data");
        db.close();
        return count;
    }

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
        db.insert("tb_data", null, values);

        db.close();
    }

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
        db.update("tb_data", values, "_id=?", new String[]{String.valueOf(id)});
        db.close();
    }

    //    public void update(String item, int price) {
//        SQLiteDatabase db = getWritableDatabase();
//        // 입력한 항목과 일치하는 행의 가격 정보 수정
//        db.execSQL("UPDATE MONEYBOOK SET price=" + price + " WHERE item='" + item + "';");
//        db.close();
//    }
//
    public void delete(int item) throws JSONException {
        JSONObject data = getData(item);
        String rawfile = data.getString("raw_file_path");

        File file = context.getFileStreamPath(rawfile);

        if (file.exists()) {
            file.delete();
        }
        // 입력한 항목과 일치하는 행 삭제
        SQLiteDatabase db = getWritableDatabase();
        db.execSQL("DELETE FROM " + "tb_data" + " WHERE _id= " + item + ";");
        db.close();
    }

    public JSONObject getData(int id) {
        Log.i("ID", String.valueOf(id));
        SQLiteDatabase db = getReadableDatabase();
        String query = "SELECT * FROM tb_data WHERE _id = ?";
        Cursor cursor = db.rawQuery(query, new String[]{String.valueOf(id)});
        cursor.moveToPosition(0);

        int totalColumn = cursor.getColumnCount();
        JSONObject rowObject = new JSONObject();

        for (int i = 0; i < totalColumn; i++) {
//            Log.i("TAG_NAME", cursor.getString(i));
            if (cursor.getColumnName(i) != null) {
                try {
                    if (cursor.getString(i) != null) {
//                        Log.d("TAG_NAME", cursor.getColumnName(i));
                        rowObject.put(cursor.getColumnName(i), cursor.getString(i));
                    } else {
                        rowObject.put(cursor.getColumnName(i), "");
                    }
                } catch (Exception e) {
//                    Log.d("TAG_NAME", e.getMessage());
                }
            }
        }
//        Log.i("object", rowObject.toString());
        cursor.close();
        db.close();
        return rowObject;
    }

    public JSONArray getResult(int start_index, int number_of_data) {
        // 읽기가 가능하게 DB 열기
        SQLiteDatabase db = getReadableDatabase();
        String result = "";

        // DB에 있는 데이터를 쉽게 처리하기 위해 Cursor를 사용하여 테이블에 있는 모든 데이터 출력
        Cursor cursor = db.rawQuery("SELECT * FROM " + "tb_data", null);
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
//                            Log.d("TAG_NAME", cursor.getString(i));
                            rowObject.put(cursor.getColumnName(i), cursor.getString(i));
                        } else {
                            rowObject.put(cursor.getColumnName(i), "");
                        }
                    } catch (Exception e) {
//                        Log.d("TAG_NAME", e.getMessage());
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

    public JSONArray getRecentData(int number_of_data) {
        // 읽기가 가능하게 DB 열기
        SQLiteDatabase db = getReadableDatabase();
        String result = "";

        // DB에 있는 데이터를 쉽게 처리하기 위해 Cursor를 사용하여 테이블에 있는 모든 데이터 출력
        Cursor cursor = db.rawQuery("SELECT * FROM " + "tb_data", null);
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
//                            Log.d("TAG_NAME", cursor.getString(i));
                            rowObject.put(cursor.getColumnName(i), cursor.getString(i));
                        } else {
                            rowObject.put(cursor.getColumnName(i), "");
                        }
                    } catch (Exception e) {
//                        Log.d("TAG_NAME", e.getMessage());
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

    public void removeAll() {
        SQLiteDatabase db = getReadableDatabase(); // helper is object extends SQLiteOpenHelper
//        db.delete("tb_data", null, null);
        db.execSQL("DROP TABLE " + "tb_data");
        db.execSQL("CREATE TABLE IF NOT EXISTS " + "tb_data" + " (_id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                "time DATETIME DEFAULT (datetime('now','localtime')), patient_name VARCHAR2(100), patient_age INT, patient_gender INT, " +
                "patient_number INT, patient_time INT, ave_speed DOUBLE, physiological_age DOUBLE, fraility_index DOUBLE," +
                "raw_file_path VARCHAR2(100));");

        File[] files = context.getFilesDir().listFiles();
        if (files != null)
            for (File file : files) {
                file.delete();
            }
    }

}
```

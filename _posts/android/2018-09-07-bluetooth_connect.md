---
layout: post
title: '[Android] Bluetooth connect and receive data (블루투스 연결 및 데이터 수신)'
tags: [android]
description: >
    Android 디바이스에서 bluetooth 연결 및 데이터 수신 방법
    (How to connect bluetooth device and receive data in android device)
---

# 1. 블루투스 관련 권한 설정

안드로이드 디바이스에서 블루투스 관련 기능을 사용하기 위해서는 블루투스 관련 권한 설정을 해야한다.

```
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<uses-permission android:name="android.permission.BLUETOOTH" />
```

안드로이드 디바이스에서 블루투스 권한을 얻기 위해서는 `AndroidManifest.xml` 파일에 위의 코드 두줄을 추가해 줘야 한다.
이 글에서는 다루지 않고 따로 정리하겠지만 블루투스 연결뿐만 아니라 주변에 있는 블루투스 기기 검색을 위해서는 다음과 같이 `ACCESS_COARSE_LOCATION` 와 `ACCESS_FINE_LOCATION` 에 대한 권한도 필요하다.

```
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```


# 2. 블루투스 활성화 및 연결

맴버변수

```
private static final int REQUEST_ENABLE_BT = 3;
public BluetoothAdapter mBluetoothAdapter = null;
Set<BluetoothDevice> mDevices;
int mPairedDeviceCount;
BluetoothDevice mRemoteDevice;
BluetoothSocket mSocket;
InputStream mInputStream;
OutputStream mOutputStream;
```



```
public void CheckBluetooth(){

    mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
    if (mBluetoothAdapter == null) {
    // 장치가 블루투스 지원하지 않는 경우
    if (activity.fragment_state >= 1)
        Toast.makeText(activity, R.string.bluetooth_availble_no, Toast.LENGTH_SHORT).show();
    } else {
    // 장치가 블루투스 지원하는 경우
        if (!mBluetoothAdapter.isEnabled()) {
        // 블루투스를 지원하지만 비활성 상태인 경우
        // 블루투스를 활성 상태로 바꾸기 위해 사용자 동의 요첨
            Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
            activity.startActivityForResult(enableBtIntent, REQUEST_ENABLE_BT);
        } else {
        // 블루투스를 지원하며 활성 상태인 경우
        // 페어링된 기기 목록을 보여주고 연결할 장치를 선택.
            selectDevice();
        }
    }
}
```


```
private void selectDevice() {
    //페어링되었던 기기 목록 획득
    mDevices = mBluetoothAdapter.getBondedDevices();
    //페어링되었던 기기 갯수
    mPairedDeviceCount = mDevices.size();
    //Alertdialog 생성(activity에는 context입력)
    AlertDialog.Builder builder = new AlertDialog.Builder(activity);
    //AlertDialog 제목 설정
    builder.setTitle(R.string.select_device_title);


    // 페어링 된 블루투스 장치의 이름 목록 작성
    final List<String> listItems = new ArrayList<String>();
    for (BluetoothDevice device : mDevices) {
        listItems.add(device.getName());
    }
    if(listItems.size() == 0){
        //no bonded device => searching
        Log.d("Bluetooth", "No bonded device");
    }else{
        Log.d("Bluetooth", "Find bonded device");
        // 취소 항목 추가
        listItems.add("Cancel");

        final CharSequence[] items = listItems.toArray(new CharSequence[listItems.size()]);
        builder.setItems(items, new DialogInterface.OnClickListener() {
            //각 아이템의 click에 따른 listener를 설정
            @Override
            public void onClick(DialogInterface dialog, int which) {
                Dialog dialog_ = (Dialog) dialog;
                // 연결할 장치를 선택하지 않고 '취소'를 누른 경우
                if (which == listItems.size()-1) {
                    Toast.makeText(dialog_.getContext(), R.string.ask_select_device, Toast.LENGTH_SHORT).show();
                
                } else {
                    //취소가 아닌 디바이스를 선택한 경우 해당 기기에 연결
                    작(items[which].toString());
                }

            }
        });

        builder.setCancelable(false);    // 뒤로 가기 버튼 사용 금지
        AlertDialog alert = builder.create();
        alert.show();   //alert 시작
    }
}
```

```
private void connectToSelectdDevice(final String selectedDeviceName) {

    //블루투스 기기에 연결하는 과정이 시간이 걸리기 때문에 그냥 함수로 수행을 하면 GUI에 영향을 미친다
    //따라서 연결 과정을 thread로 수행하고 thread의 수행 결과를 받아 다음 과정으로 넘어간다.
    final Handler mHandler = new Handler() {
        public void handleMessage(Message msg) {
            if (msg.what == 1) // 연결 완료
            {
                try {
                    //연결이 완료되면
                    mOutputStream = mSocket.getOutputStream();
                    mInputStream = mSocket.getInputStream();
                    // 데이터 수신 준비
                    bluetooth_connect_flag = true;
                    beginListenForData();
                    
                } catch (IOException e) {
                    e.printStackTrace();
                }
            } else {    //연결 실패
                Toast.makeText(activity,R.string.warning_no_device, Toast.LENGTH_SHORT).show();
                try {
                    mSocket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    };

    //연결과정을 수행할 thread 생성
    Thread thread = new Thread(new Runnable() {
        public void run() {
            //선택된 기기의 이름을 갖는 bluetooth device의 object
            mRemoteDevice = getDeviceFromBondedList(selectedDeviceName);
            UUID uuid = UUID.fromString("00001101-0000-1000-8000-00805f9b34fb");

            try {
                // 소켓 생성
                mSocket = mRemoteDevice.createRfcommSocketToServiceRecord(uuid);
                // RFCOMM 채널을 통한 연결
                mSocket.connect();
                connected_bluetooth_device_name = selectedDeviceName;
                mHandler.sendEmptyMessage(1);
            } catch (Exception e) {
                // 블루투스 연결 중 오류 발생
                mHandler.sendEmptyMessage(-1);
            }
        }
    });
    thread.start();
}

//입력값에 해당하는 bluetooth device object 출력
BluetoothDevice getDeviceFromBondedList(String name) {
    BluetoothDevice selectedDevice = null;
    mDevices = mBluetoothAdapter.getBondedDevices();
    //pair 목록에서 해당 이름을 갖는 기기 검색, 찾으면 해당 device 출력
    for (BluetoothDevice device : mDevices) {
        if (name.equals(device.getName())) {
            selectedDevice = device;
            break;
        }
    }
    return selectedDevice;
}


```


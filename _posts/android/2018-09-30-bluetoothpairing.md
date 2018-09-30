---
layout: post
title: '[Android] Android  bluetooth pairing in code (안드로이드 블루투스 페어링)'
tags: [android]
description: >
    How to pairing bluetooth device
---

이 글에서는 안드로이드 어플에서 직접 주변에 있는 기기들을 검색하고 pairing을 수행하는 방법에 대해서 설명한다. 

[이전 글](http://jinyongjeong.github.io/2018/09/29/bluetoothconnect/)에서는 페어링이 되어있는 기기 목록에서 사용자가 선택하여 기기에 연결하는 방법에 대해서 설명하였다. 

이 글에서 사용된 블루투스 연결 기기는 일반적으로 많이 사용되는 HC-06 블루투스 모듈이다. 


## 1. 블루투스 관련 권한 설정

안드로이드 디바이스에서 블루투스 관련 기능을 사용하기 위해서는 블루투스 관련 권한 설정을 해야한다.
또한 블루투스 권한설정 뿐만 아니라 주변 기기의 검색을 위해서는 위치 접근에 대한 권한이 필요하다. 따라서 다음과 같이 4개에 대해서 접근 권한을 설정한다.

```
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```
안드로이드 디바이스에서 블루투스 권한을 얻기 위해서는 `AndroidManifest.xml` 파일에 위의 코드 두줄을 추가해 줘야 한다.

## 2. 블루투스 기기 검색에 대한 IntentFilter 설정

안드로이드의 IntentFilter는 안드로이드 시스템이나 다른 어플에서 broadcast하는 메세지를 받을 수 있도록 설정하는 것이다. 즉 어떤 메세지를 날렸을 때 받을 것인지를 설정하는 것이다. 아래 코드는 블루투스의 상태변화, 연결됨, 연결 끊어짐 등에 대한 설정 뿐만 아니라 기기 검색 시작, 종료, 검색됨 등에 대한 메세지도 받을 수 있도록 설정하는 코드이다. 

```
IntentFilter stateFilter = new IntentFilter();
stateFilter.addAction(BluetoothAdapter.ACTION_STATE_CHANGED); //BluetoothAdapter.ACTION_STATE_CHANGED : 블루투스 상태변화 액션
stateFilter.addAction(BluetoothAdapter.ACTION_CONNECTION_STATE_CHANGED);
stateFilter.addAction(BluetoothDevice.ACTION_ACL_CONNECTED); //연결 확인
stateFilter.addAction(BluetoothDevice.ACTION_ACL_DISCONNECTED); //연결 끊김 확인
stateFilter.addAction(BluetoothDevice.ACTION_BOND_STATE_CHANGED);
stateFilter.addAction(BluetoothDevice.ACTION_FOUND);    //기기 검색됨
stateFilter.addAction(BluetoothAdapter.ACTION_DISCOVERY_STARTED);   //기기 검색 시작
stateFilter.addAction(BluetoothAdapter.ACTION_DISCOVERY_FINISHED);  //기기 검색 종료
stateFilter.addAction(BluetoothDevice.ACTION_PAIRING_REQUEST);
registerReceiver(mBluetoothStateReceiver, stateFilter);
```

`registerReceiver`는 intentfilter를 특정 broadcast receiver로 연결시켜주는 역할을 한다. 이 코드에서는 `mBluetoothStateReceiver`라는 broadcast receiver에 연결시켜 주었다. 만약 앱이 비활성화 상태일 때 broadcast를 수신하지 않도록 하기 위해서는 `onPause`와 같은 함수에서 unregister를 해주어야 한다. 

```
unregisterReceiver(bluetoothRequest.mBluetoothStateReceiver);
```

## 3. Broadcast Receiver 정의

다음으로는 Intentfilter에 의해서 메세지를 수신할 broadcast receiver를 정의해야 한다. 


```
BroadcastReceiver mBluetoothStateReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        final String action = intent.getAction();   //입력된 action
        Toast.makeText(context, "받은 액션 : "+action , Toast.LENGTH_SHORT).show();
        Log.d("Bluetooth action", action);
        final BluetoothDevice device =   intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
        String name = null;
        if (device != null) {
            name = device.getName();    //broadcast를 보낸 기기의 이름을 가져온다.
        }
        //입력된 action에 따라서 함수를 처리한다
        switch (action){
            case BluetoothAdapter.ACTION_STATE_CHANGED: //블루투스의 연결 상태 변경
                final int state = intent.getIntExtra(BluetoothAdapter.EXTRA_STATE, BluetoothAdapter.ERROR);
                switch(state) {
                    case BluetoothAdapter.STATE_OFF:

                        break;
                    case BluetoothAdapter.STATE_TURNING_OFF:

                        break;
                    case BluetoothAdapter.STATE_ON:

                        break;
                    case BluetoothAdapter.STATE_TURNING_ON:

                        break;
                }

                break;
            case BluetoothDevice.ACTION_ACL_CONNECTED:  //블루투스 기기 연결

                break;
            case BluetoothDevice.ACTION_BOND_STATE_CHANGED:

                break;
            case BluetoothDevice.ACTION_ACL_DISCONNECTED:   //블루투스 기기 끊어짐
                
                break;
                
            case BluetoothAdapter.ACTION_DISCOVERY_STARTED: //블루투스 기기 검색 시작
                
                break;
            case BluetoothDevice.ACTION_FOUND:  //블루투스 기기 검색 됨, 블루투스 기기가 근처에서 검색될 때마다 수행됨
                String device_name = device.getName();
                String device_Address = device.getAddress();
                //본 함수는 블루투스 기기 이름의 앞글자가 "GSM"으로 시작하는 기기만을 검색하는 코드이다
                if(device_name != null && device_name.length() > 4){
                    Log.d("Bluetooth Name: ", device_name);
                    Log.d("Bluetooth Mac Address: ", device_Address);
                    if(device_name.substring(0,3).equals("GSM")){
                        bluetooth_device.add(device);
                    }
                }
                break;
            case BluetoothDevice.ACTION_DISCOVERY_FINISHED:    //블루투스 기기 검색 종료
                Log.d("Bluetooth", "Call Discovery finished");
                StartBluetoothDeviceConnection();   //원하는 기기에 연결
                break;
            case BluetoothDevice.ACTION_PAIRING_REQUEST:

                break;
            }

        }
    };
```

위와 같이 주변의 블루투스 기기를 검색해서 처리하기 위한 broadcast receiver를 등록하였다. 

주변에 모든 블루투스 기기를 모두 검색하기 위해서는 약 11초 전후의 시간이 걸리며 action의 순서는 다음과 같다. 

사용자 검색 요청 ->   `BluetoothAdapter.ACTION_DISCOVERY_STARTED` -> `BluetoothDevice.ACTION_FOUND` (검색되는 기기 갯수대로) -> `BluetoothDevice.ACTION_DISCOVERY_FINISHED`

따라서 검색되는 기기에 대한 저장을 한 후에 `BluetoothDevice.ACTION_DISCOVERY_FINISHED` 가 수행되었을 때 원하는 기기에 접근해서 페어링을 수행하면 된다. 


## 4. 블루투스 기기 검색 요청

블루투스 기기에 대한 요청을 하기 위해서는 bluetooth adapter 객체를 가지고 있어야 한다. 

이전 블루투스 연결 글에서 처럼 다음과 같은 맴버 변수가 있으며 다음과 같이 검색 수행을 시작시킬 수 있다.

```
private static final int REQUEST_ENABLE_BT = 3;
public BluetoothAdapter mBluetoothAdapter = null;
Set<BluetoothDevice> mDevices;
int mPairedDeviceCount;
BluetoothDevice mRemoteDevice;
BluetoothSocket mSocket;
InputStream mInputStream;
OutputStream mOutputStream;
Thread mWorkerThread;
int readBufferPositon;      //버퍼 내 수신 문자 저장 위치
byte[] readBuffer;      //수신 버퍼
byte mDelimiter = 10;
```

```
mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter();   //블루투스 adapter 획득
mBluetoothAdapter.startDiscovery(); //블루투스 기기 검색 시작
```

`mBluetoothAdapter.startDiscovery()`가 실행되면 블루투스 기기 검색이 시작되며 broadcast receiver로 메세지가 들어오게 된다. 

이제 검색이 시작되었으니 검색이 종료되었을 때, 즉 `BluetoothDevice.ACTION_DISCOVERY_FINISHED`이 수행될 때 검색된 목록 중에서

원하는 기기로 연결을 하면 된다. 

본 예제에서는 검색 종료시 `StartBluetoothDeviceConnection()` 함수가 수행되었다. 


## 5. 검색된 블루투스 기기에 페어링 및 연결

앞에서 검색된 기기의 이름은 `bluetooth_device`라는 맴버변수에 저장되어 있다. 이제 이 리스트 중에서 연결한 기기를 사용자가 선택할 수 있도록 해준다. 

그리고 사용자가 선택할 경우 그 기기에 연결을 수행한다. 

```
public void StartBluetoothDeviceConnection(){

//Bluetooth connection start

    if(bluetooth_device.size() == 0){
        Toast.makeText(activity,"There is no device", Toast.LENGTH_SHORT).show();
    }
   
    AlertDialog.Builder builder = new AlertDialog.Builder(activity);
    builder.setTitle(R.string.select_device_title);
 
    // 페어링 된 블루투스 장치의 이름 목록 작성
    List<String> listItems = new ArrayList<String>();
    for (BluetoothDevice bt_device : bluetooth_device) {
    listItems.add(bt_device.getName());
    }
    listItems.add("Cancel");    // 취소 항목 추가

    final CharSequence[] items = listItems.toArray(new CharSequence[listItems.size()]);

    builder.setItems(items, new DialogInterface.OnClickListener() {
    @Override
    public void onClick(DialogInterface dialog, int which) {
        Dialog dialog_ = (Dialog) dialog;
        
        if (which == bluetooth_device.size()) {
            // 연결할 장치를 선택하지 않고 '취소'를 누른 경우

        } else {
            // 기기 이름을 선택한 경우 선택한 기기 이름과 같은 블루투스 객체를 찾아서 연결을 시도한다
            for (BluetoothDevice bt_device : bluetooth_device) {
                if (bt_device.getName().equals(items[which].toString())) {
                    Log.d("Bluetooth Connect", bt_device.getName());
                    ConnectBluetoothDevice(bt_device);  //해당하는 블루투스 객체를 이용하여 연결 시도
                    Log.d("Bluetooth Connect", "ConnectBluetoothDevice");
                    break;
                }
            }
        }

        }
    });
    builder.setCancelable(false);    // 뒤로 가기 버튼 사용 금지
    AlertDialog alert = builder.create();
    alert.show();
    Log.d("Bluetooth Connect", "alert start");
}
```

## 6. 해당 기기에 연결

이제 사용자가 선택한 블루투스 객체를 이용해서 해당 블루투스 기기에 연결 혹은 pairing을 수행해야 한다. 

다음 함수는 이러한 역할을 수행한다.

```
public void ConnectBluetoothDevice(final BluetoothDevice device){

    mDevices = mBluetoothAdapter.getBondedDevices();
    mPairedDeviceCount = mDevices.size();

    //pairing되어 있는 기기의 목록을 가져와서 연결하고자 하는 기기가 이전 기기 목록에 있는지 확인
    boolean already_bonded_flag = false;
    if(mPairedDeviceCount > 0){
        for (BluetoothDevice bonded_device : mDevices) {
            if(activity.bluetooth_device_name.equals(bonded_device.getName())){
                already_bonded_flag = true;
            }
        }
    }
    //pairing process
    //만약 pairing기록이 있으면 바로 연결을 수행하며, 없으면 createBond()함수를 통해서 pairing을 수행한다.
    if(!already_bonded_flag){
        try {
            //pairing수행
            device.createBond();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }else{
        connectToSelectedDevice(device);
    }
}
```

위 코드처럼 이전 연결 목록을 확인하여 처음 연결되는 기기는 `createBond()`함수를 통해 페어링을 수행하며, 처음 연결이 아닌 경우에는 바로 연결을 수행한다.

연결을 수행하기 위한 함수인 `connectToSelectedDevice`는 [이전 글](http://jinyongjeong.github.io/2018/09/29/bluetoothconnect/)에서 확인할 수 있다.

물론 이러한 블루투스에 연결되는 과정들은 시간이 소요되는 과정들이기 때문에 연결 중에는 로딩화면을 띄워주면 좋다. 

위의 코드에서는 로딩 화면을 띄워주는 부분은 포함하고 있지 않다.



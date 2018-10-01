---
layout: post
title: '[Android] Android bluetooth connect and receive data(안드로이드 블루투스 연결 및 데이터 수신)'
tags: [android]
description: >
    How to connect bluetooth device and receive data in android device
---



이 글은 일반적으로 많이 사용되는 HC-06 블루투스 모듈과 통신하기 위한 방법을 설명한다. HC-06은 블루투스 2.0을 이용하며 최근 많이 사용되고 있는 저전력 블루투스 모듈인 BLE와는 다르다.


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
블루투스 연결 관련 함수들을 class 로 작성해서 사용하는 것이 좋은 방법이며 다음과 같은 맴버 변수를 선언하였다.

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

가장 먼저 사용하는 기기가 블루투스를 지원하는지, 지원한다면 블루투스가 켜져있는지 확인해야 한다. 

```
public void CheckBluetooth(){

    mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
	if (mBluetoothAdapter == null) {
		// 장치가 블루투스 지원하지 않는 경우
		Toast.makeText(activity, "Bluetooth no available", Toast.LENGTH_SHORT).show();
	} else {
		// 장치가 블루투스 지원하는 경우
		if (!mBluetoothAdapter.isEnabled()) {
			// 블루투스를 지원하지만 비활성 상태인 경우
			// 블루투스를 활성 상태로 바꾸기 위해 사용자 동의 요첨
			Intent enableBtIntent = new Intent( BluetoothAdapter.ACTION_REQUEST_ENABLE);
			activity.startActivityForResult(enableBtIntent, REQUEST_ENABLE_BT);
		} else {
			// 블루투스를 지원하며 활성 상태인 경우
			// 페어링된 기기 목록을 보여주고 연결할 장치를 선택.
			selectDevice();
		}
	}
}
```

이상이 없는 경우에 pairing 목록에서 연결하고자 하는 기기를 검색해서 사용자에게 선택하도록 한다. 이때 bluetooth 모듈에 연결하기 위해서는 기기와 paring 한 기록이 있어야 한다. paring이 되어 있다는 의미는 기기의 정보가 핸드폰에 등록이 되어 있으며 서로 존재를 알고 있다는 의미이다. pairing은 핸드폰의 블루투스 설정에서 기기 검색을 통해 가능하다.  (코드를 통해서 pairing 과정을 수행할 수도 있다. 하지만 이 글에서는 페어링이 되어 있는 기기에 연결하는 과정만을 설명한다)
`selectDevice()` 함수는 현재 핸드폰에 등록되어 있는 pairing목록을 화면의 띄워서 사용자에게 연결하고자 하는 기기를 선택할 수 있도록 화면을 띄워준다.

```
private void selectDevice() {
    //페어링되었던 기기 목록 획득
    mDevices = mBluetoothAdapter.getBondedDevices();
    //페어링되었던 기기 갯수
    mPairedDeviceCount = mDevices.size();
    //Alertdialog 생성(activity에는 context입력)
    AlertDialog.Builder builder = new AlertDialog.Builder(activity);
    //AlertDialog 제목 설정
    builder.setTitle("Select device");


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
                    Toast.makeText(dialog_.getContext(), "Choose cancel", Toast.LENGTH_SHORT).show();
                
                } else {
                    //취소가 아닌 디바이스를 선택한 경우 해당 기기에 연결
                    connectToSelectedDevice(items[which].toString());
                }

            }
        });

        builder.setCancelable(false);    // 뒤로 가기 버튼 사용 금지
        AlertDialog alert = builder.create();
        alert.show();   //alert 시작
    }
}
```

`connectToSelectedDevice()` 함수는 선택된 기기로 연결하는 과정을 시작하는 함수이다.

```
private void connectToSelectedDevice(final String selectedDeviceName) {
    //블루투스 기기에 연결하는 과정이 시간이 걸리기 때문에 그냥 함수로 수행을 하면 GUI에 영향을 미친다
    //따라서 연결 과정을 thread로 수행하고 thread의 수행 결과를 받아 다음 과정으로 넘어간다.
    
    //handler는 thread에서 던지는 메세지를 보고 다음 동작을 수행시킨다.
    final Handler mHandler = new Handler() {
        public void handleMessage(Message msg) {
            if (msg.what == 1) // 연결 완료
            {
                try {
                    //연결이 완료되면 소켓에서 outstream과 inputstream을 얻는다. 블루투스를 통해
                    //데이터를 주고 받는 통로가 된다.
                    mOutputStream = mSocket.getOutputStream();
                    mInputStream = mSocket.getInputStream();
                    // 데이터 수신 준비
                    beginListenForData();
                    
                } catch (IOException e) {
                    e.printStackTrace();
                }
            } else {    //연결 실패
                Toast.makeText(activity,"Please check the device", Toast.LENGTH_SHORT).show();
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
                // RFCOMM 채널을 통한 연결, socket에 connect하는데 시간이 걸린다. 따라서 ui에 영향을 주지 않기 위해서는
                // Thread로 연결 과정을 수행해야 한다.
                mSocket.connect();
                mHandler.sendEmptyMessage(1);
            } catch (Exception e) {
                // 블루투스 연결 중 오류 발생
                mHandler.sendEmptyMessage(-1);
            }
        }
    });
    
    //연결 thread를 수행한다
    thread.start();
}

//기기에 저장되어 있는 해당 이름을 갖는 블루투스 디바이스의 bluetoothdevice 객채를 출력하는 함수
//bluetoothdevice객채는 기기의 맥주소뿐만 아니라 다양한 정보를 저장하고 있다.

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

socket에 연결이 완료되면 데이터를 수신할 thread (`beginListenForDat()`) 를 실행한다.

```
//블루투스 데이터 수신 Listener
protected void beginListenForData() {
    final Handler handler = new Handler();
    readBuffer = new byte[1024];  //  수신 버퍼
    readBufferPositon = 0;        //   버퍼 내 수신 문자 저장 위치
    
    // 문자열 수신 쓰레드
    mWorkerThread = new Thread(new Runnable() {
        @Override
        public void run() {

            while (!Thread.currentThread().isInterrupted()) {

                try {

                    int bytesAvailable = mInputStream.available();
                    if (bytesAvailable > 0) { //데이터가 수신된 경우
                        byte[] packetBytes = new byte[bytesAvailable];
                        mInputStream.read(packetBytes);
                        for (int i = 0; i < bytesAvailable; i++) {
                            byte b = packetBytes[i];
                            if (b == mDelimiter) {
                                byte[] encodedBytes = new byte[readBufferPositon];
                                System.arraycopy(readBuffer, 0, encodedBytes, 0, encodedBytes.length);
                                final String data = new String(encodedBytes, "US-ASCII");
                                readBufferPositon = 0;
                                handler.post(new Runnable() {
                                    public void run() {
                                        //수신된 데이터는 data 변수에 string으로 저장!! 이 데이터를 이용하면 된다.
                                    
                                        char[] c_arr = data.toCharArray(); // char 배열로 변환
                                        if (c_arr[0] == 'a') {
                                            if (c_arr[1] == '1') {
                                            
                                            //a1이라는 데이터가 수신되었을 때
                                            
                                            }
                                            if (c_arr[1] == '2') {
                                            
                                            //a2라는 데이터가 수신 되었을 때
                                            }
                                        }
                                    }
                                });
                            } else {
                            readBuffer[readBufferPositon++] = b;
                            }
                        }
                    }
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    });
    //데이터 수신 thread 시작
    mWorkerThread.start();
}
```

마지막으로 데이터를 전송하는 방법은 다음과 같다.

```
public void SendResetSignal(){
    String msg = "bs00000";
    try {
        mOutputStream.write(msg.getBytes());
    } catch (IOException e) {
        e.printStackTrace();
    }
}

```

이번 글에서는 pairing목록에서 기기 정보를 가져와서 블루투스 통신을 수행하는 방법에 대해서 다루었다. 다음에는 기기가 pairing목록에 없을 때
기기를 검색해서 pairing과정까지 한번에 수행하는 방법을 알아볼 것이다. 

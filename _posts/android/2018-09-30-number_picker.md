---
layout: post
title: '[Android] Android number picker with dialog (안드로이드 dialog를 이용한 number picker 입력 받기)'
tags: [android]
description: >
    number picker with dialog (dialog를 이용한 number picker 입력받기)
---

이번 글에서는 사용자에게 숫자를 입력받기 위한 방법중에서 number picker를 사용하는 방법에 대해서 알아본다.

숫자를 입력받는 방법으로는 edittext에 직접 키보드로 숫자 입력을 받는 방법이 있다. 

하지만 앱을 단순하고 유저 관점에서 사용성이 좋게 하기 위해서는 직접 입력하는 것보다 클릭으로 가능하도록 처리하는 것이 좋다.

따라서 숫자를 아래 그림과 같이 직접 입력이 아닌 입력 가능한 숫자 중에서 선택하도록 만드는 것이 좋다.

<img src="/images/post/android/2018-09-03-number_picker/number_picker.jpeg" width="70%" style="margin-left: auto; margin-right: auto; display: block;"/>


위 예시는 특정 값에 해당하는 설정 버튼을 눌렀을 때 숫자를 선택할 수 있는 dialog가 팝업으로 뜨고 사용자가 숫자를 선택 후 확인을 누르면 적용이 되는 방식이다. 

이러한 코드를 구현해 보자.


## 1. NumberpickerDialog 클래스 구현

아래 코드는 NumberpickrDialog를 구현한 클래스이다. 각 코드마다 기능들은 주석으로 추가해 놓았다. 


```
import android.app.AlertDialog;
import android.app.Dialog;
import android.app.DialogFragment;
import android.content.DialogInterface;
import android.os.Bundle;
import android.util.Log;
import android.widget.NumberPicker;

public class NumberPickerDialog extends DialogFragment {
    private NumberPicker.OnValueChangeListener valueChangeListener;

    String title;	//dialog 제목
    String subtitle;	//dialog 부제목
    int minvalue;	//입력가능 최소값
    int maxvalue;	//입력가능 최대값
    int step;	//선택가능 값들의 간격
    int defvalue;	//dialog 시작 숫자 (현재값)

    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {

        final NumberPicker numberPicker = new NumberPicker(getActivity());

		 //Dialog 시작 시 bundle로 전달된 값을 받아온다
		 
        title = getArguments().getString("title");
        subtitle = getArguments().getString("subtitle");
        minvalue = getArguments().getInt("minvalue");
        maxvalue = getArguments().getInt("maxvalue");
        step = getArguments().getInt("step");
        defvalue = getArguments().getInt("defvalue");

		 //최소값과 최대값 사이의 값들 중에서 일정한 step사이즈에 맞는 값들을 배열로 만든다.
        String[] myValues = getArrayWithSteps(minvalue, maxvalue, step);
        
        //displayedvalues를 사용하지 않고 min/max 값을 설정해서 선택을 받을 때는 선택한 보여지는 값이 바로 리턴되는 값이다. 
        //하지만 이런경우에는 보여지는 값은 최소값과 최대값사이에 값이 1씩 증가되는 값들이 모두 보여지게 된다. (별도의 step을 줄 수 없다)
        //일정한 간격의 숫자만을 선택할 수 있게 하려면, String 배열에 display가 되는 값들을 입력해서 setDisplayedValues함수로 입력해줘야 하며
        //이런경우 그 숫자가 리턴값이 아닌 배열의 인덱스가 리턴값이 된다. 따라서 minValue와 maxvalue는 이에 맞게 설정해 주어야 한다.
        
        numberPicker.setMinValue(0);
        numberPicker.setMaxValue((maxvalue - minvalue) / step);
        numberPicker.setDisplayedValues(myValues);
        
        //현재값 설정 (dialog를 실행했을 때 시작지점)
        numberPicker.setValue((defvalue - minvalue) / step);
        //키보드 입력을 방지
        numberPicker.setDescendantFocusability(NumberPicker.FOCUS_BLOCK_DESCENDANTS);

        AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
        //제목 설정
        builder.setTitle(title);
        //부제목 설정
        builder.setMessage(subtitle);

		 //Ok button을 눌렀을 때 동작 설정
        builder.setPositiveButton("OK", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
            
				 //dialog를 종료하면서 값이 변했다는 함수는 onValuechange함수를 실행시킨다. 
				 //실제 구현에서는 이 클레스의 함수를 재 정의해서 동작을 수행한다.
				 
                valueChangeListener.onValueChange(numberPicker,
                        numberPicker.getValue(), numberPicker.getValue());
            }
        });

		 //취소 버튼을 눌렀을 때 동작 설정
        builder.setNegativeButton("CANCEL", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {

            }
        });

        builder.setView(numberPicker);
		//number picker 실행
        return builder.create();
    }

	//Listener의 getter
    public NumberPicker.OnValueChangeListener getValueChangeListener() {
        return valueChangeListener;
    }

	//Listener의 setter
    public void setValueChangeListener(NumberPicker.OnValueChangeListener valueChangeListener) {
        this.valueChangeListener = valueChangeListener;
    }

	//최소값부터 최대값가지 일정 간격의 값을 String 배열로 출력
    public String[] getArrayWithSteps(int min, int max, int step) {

        int number_of_array = (max - min) / step + 1;

        String[] result = new String[number_of_array];

        for (int i = 0; i < number_of_array; i++) {
            result[i] = String.valueOf(min + step * i);
        }
        return result;
    }
}
```

이 코드에서 주의해야 할 점은 다음과 같다. 

### 1. 일정 간격의 입력값을 입력 후보군으로 보여주고 싶을 때는 setDisplayedValues 함수를 사용한다

일반적인 numberpicker는 최소값부터 최대값까지 1의 간격의 숫자를 보여주고 선택할 수 있다. 이런 경우에는 Dialog에서 사용자가 선택한 값이 바로 리턴값이 된다. 
하지만 최소값부터 최대값까지 일정한 step 간격의 숫자를 보여주고 선택하게 하고 싶다면 (맨 위에 그림처럼) setDisplayedValues함수를 이용해야 한다. 
setDisplayedValues 함수는 사용자에게 보여주고자 하는 숫자들을 String 배열로 입력받는다. Dialog는 입력받은 숫자들의 String 값을 사용자에게 보여준다. 
이때 주의해야 할 점은 사용자가 선택한 숫자에 대한 리턴값은 String 배열의 index이다. 즉 리턴을 받았을 때 실제 사용자가 선택한 값은 index를 이용해서 다시 계산해야 한다는 것이다. 
또한 이때 minvalue와 maxvalue또한 String 배열의 index에 맞게 설정해 줘야 한다. 

### 2. 외부 클레스에서 implements를 이용해서 Listener를 등록해서 사용한다

사용자가 숫자를 선택하고 적용을 시키기 위해 `OK` 버튼을 누르면 `valueChangeListener.onValueChange(numberPicker,numberPicker.getValue(), numberPicker.getValue())` 함수가 실행된다. 이 함수는 외부에서 implement로 상속한 함수의 onValueChange함수를 실행시킨다. 이와같은 역할을 하는 코드는 다음과 같다. 

```
//구현하고자 하는 fragment는 implements로 NumberPicker.OnValueChangeListener 를 받는다
public class FragmentDeviceSetting extends Fragment implements NumberPicker.OnValueChangeListener{

	//중간 내용 생략


	//valueChangeListener.onValueChange가 수행되었을 때 수행될 함수
    @Override
    public void onValueChange(NumberPicker numberPicker, int i, int i1) {
		int setting_value = activity.sensor_minimum_distance + numberPicker.getValue()*activity.distance_setting_step;

		//원하는 동작 수행
	}

	//Dialog를 시작하는 함수, 특정 버튼을 눌렀을 때 이 함수를 실행 시키면 된다
    public void showNumberPicker(View view, String title, String subtitle, int maxvalue, int minvalue, int step, int defvalue){
        NumberPickerDialog newFragment = new NumberPickerDialog();

		//Dialog에는 bundle을 이용해서 파라미터를 전달한다
        Bundle bundle = new Bundle(6); // 파라미터는 전달할 데이터 개수
        bundle.putString("title", title); // key , value
        bundle.putString("subtitle", subtitle); // key , value
        bundle.putInt("maxvalue", maxvalue); // key , value
        bundle.putInt("minvalue", minvalue); // key , value
        bundle.putInt("step", step); // key , value
        bundle.putInt("defvalue", defvalue); // key , value
        newFragment.setArguments(bundle);
        //class 자신을 Listener로 설정한다
        newFragment.setValueChangeListener(this);
        newFragment.show(activity.getFragmentManager(), "number picker");
    }

}
```

위에 주석으로 실제 Dialog를 실행하는 fragment코드까지 추가하였다. 주석을 보면서 이해해보면 그렇게 어렵지 않음을 알 수 있다. 하지만 여기서 주의해야 할 점은 `numberPicker.getValue()`로 얻어지는 값이 실제 사용자에게 보여지는 값이 아닌 String 배열에 들어가 있는 값들의 index값이라는 점이다. 따라서 얻어지는 index값을 이용해서 실제 사용자가 선택한 값을 추정해야 한다. 

```
int setting_value = sensor_minimum_distance + numberPicker.getValue()* distance_setting_step;
```

위의 예제 코드에서는 String 배열을 만들때 과정을 이용해서 역으로 계산하였다. 

위의 코드들은 실제 사용하고 있는 코드에서 필요한 부분만 발췌하여 작성하였기 때문에 저 코드만으로는 동작하지 않을 수 있다. 코드를 이해하고 본인의 코드에 맞게 응용하면 된다.

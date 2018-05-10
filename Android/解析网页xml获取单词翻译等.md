首先，在金山词霸上获取到key，多余的不详细说了，这是我获取到的key：

```java
05C4F33DDAE7C18C6C24259B7C289136
```

然后调用他的免费api：

```XM
http://dict-co.iciba.com/api/dictionary.php?w=word&key=05C4F33DDAE7C18C6C24259B7C289136
```

以单词word为例，在网页上生成xml文件：

```xml
<dict num="219" id="219" name="219">
<key>word</key>
<ps>wɜ:d</ps>
<pron>...</pron>
<ps>wɜrd</ps>
<pron>...</pron>
<pos>n.</pos>
<acceptation>单词；话语；诺言；消息；</acceptation>
<pos>vt.</pos>
<acceptation>措辞，用词；用言语表达；</acceptation>
<pos>vi.</pos>
<acceptation>讲话；</acceptation>
<sent>...</sent>
<sent>
<orig>
This file is on the PDF file format into WORD document format.
</orig>
<trans>此文件是关于把PDF文件格式转换成WORD文件格式的.</trans>
</sent>
<sent>
<orig>We WORD file on a comprehensive test.</orig>
<trans>对WORD档我们进行了全方位的测试.</trans>
</sent>
<sent>
<orig>
Runs on Microsoft Word: You do not have to handle several windows on the screen.
</orig>
<trans>运行在微软Word中: 你不必在桌面上操作多个窗口.</trans>
</sent>
<sent>
<orig>
Also, make your resume available in several formats -- text only , Microsoft Word a PDF.
</orig>
<trans>另外, 要确保你的简历要有几个版本 — 纯文字 、 Word档、PDF档.</trans>
</sent>
</dict>
```

剩下的就是从这个xml中取得想要的东西了：

获取单词，首先创建单词类：（这里先只写两个参数，单词本身和音标，剩下的同理）

```java
package com.example.playenglish.bean;

/**
 * Created by 解奕鹏 on 2018/5/9.
 */

public class Word {
    private String key;
    private String ps;

    public String getKey() {
        return key;
    }

    public void setKey(String key) {
        this.key = key;
    }

    public String getPs() {
        return ps;
    }

    public void setPs(String ps) {
        this.ps = ps;
    }

    @Override
    public String toString() {
        return "key"+key+",ps"+ps;
    }
}

```

CheckActivity如下：

```java
package com.example.playenglish;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.text.TextUtils;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

import com.example.playenglish.bean.Word;
import com.example.playenglish.tools.PullWordParser;
import com.kymjs.rxvolley.RxVolley;
import com.kymjs.rxvolley.client.HttpCallback;
import com.kymjs.rxvolley.toolbox.Loger;
import com.youdao.sdk.app.Language;
import com.youdao.sdk.app.LanguageUtils;
import com.youdao.sdk.app.YouDaoApplication;
import com.youdao.sdk.ydonlinetranslate.Translator;
import com.youdao.sdk.ydtranslate.Translate;
import com.youdao.sdk.ydtranslate.TranslateErrorCode;
import com.youdao.sdk.ydtranslate.TranslateListener;
import com.youdao.sdk.ydtranslate.TranslateParameters;

import org.json.JSONArray;
import org.json.JSONObject;
import org.xmlpull.v1.XmlPullParser;
import org.xmlpull.v1.XmlPullParserException;
import org.xmlpull.v1.XmlPullParserFactory;

import java.io.IOException;
import java.io.InputStream;
import java.io.StringReader;
import java.util.List;
import java.util.logging.Logger;

import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;

public class CheckActivity extends AppCompatActivity {
    private static final String TAG = "CheckActivity";
    private String getResultUrl = null;
    private String resultUrl = null;
    private Word word = new Word();
    private Button checkBackButton;
    private Button checkSearchButton;
    private EditText edit_word;
    //用于显示所取得的翻译等信息
    private TextView tv_word;
    private TextView tv_phonetic;
    private TextView tv_trans;
    private TextView tv_source;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_check);
        init();
        inClick();
    }

    private void inClick() {
        checkBackButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                finish();
            }
        });
        checkSearchButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                getResultUrl = edit_word.getText().toString();
                if (getResultUrl == null) {
                    Toast.makeText(CheckActivity.this, "输入框不能为空", Toast.LENGTH_SHORT).show();
                } else {
                    resultUrl = "http://dict-co.iciba.com/api/dictionary.php?w=" + getResultUrl + "&key=05C4F33DDAE7C18C6C24259B7C289136";
                    sendRequestWithOkHttp();
                }

            }


        });
    }

    private void sendRequestWithOkHttp() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    OkHttpClient client = new OkHttpClient();
                    Request request = new Request.Builder()
                            .url(resultUrl)
                            .build();
                    Response response = client.newCall(request).execute();
                    String responseData = response.body().string();
                    parseXMLWithPull(responseData);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    private void parseXMLWithPull(String xmlData) {
        try {
            XmlPullParserFactory factory = XmlPullParserFactory.newInstance();
            XmlPullParser xmlPullParser = factory.newPullParser();
            xmlPullParser.setInput(new StringReader(xmlData));
            int eventType = xmlPullParser.getEventType();
            while (eventType != XmlPullParser.END_DOCUMENT) {
                Log.d(TAG, "parseXMLWithPull: ");
                String nodeName = xmlPullParser.getName();
                switch (eventType) {
                    case XmlPullParser.START_TAG:
                        Log.d(TAG, "parseXMLWithPull: " + nodeName);
                        if ("key".equals(nodeName)) {
                            word.setKey(xmlPullParser.nextText());
                        } else if ("ps".equals(nodeName)) {
                            word.setPs(xmlPullParser.nextText());
                        }
                        break;
                    case XmlPullParser.END_TAG:
                        if ("dict".equals(nodeName)) {
                            //打印结果：
                            Log.d(TAG, "parseXMLWithPull: " + word.getKey());
                            Log.d(TAG, "parseXMLWithPull: " + word.getPs());
                        }
                        break;
                    default:
                        break;
                }
                eventType = xmlPullParser.next();
            }
        } catch (Exception e) {
            Log.e(TAG, "parseXMLWithPull: ", e);
        }
    }


    private void init() {
        checkBackButton = findViewById(R.id.Back_Check_Button);
        checkSearchButton = findViewById(R.id.Search_Check_Button);
        edit_word = findViewById(R.id.Edit_Check_EditText);
        tv_word = findViewById(R.id.The_Word_TextView);
        tv_phonetic = findViewById(R.id.The_Pronunciation_TextView);
        tv_trans = findViewById(R.id.The_Translation_TextView);
        tv_source = findViewById(R.id.The_From_TextView);
    }
}
```

输出结果：


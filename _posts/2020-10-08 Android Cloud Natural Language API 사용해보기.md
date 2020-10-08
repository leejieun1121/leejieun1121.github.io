---
title : "Android Cloud Natural Language API 사용해보기 "
excerpt : "Android"

categories:
  - Android
tags :
  - Android
  - Java
 

---

## :raised_hands: Cloud Natural Language API사용하기 :raised_hands:

원래는 딥러닝을 공부해서 감성분석 모델을 직접 만들어 보는게 목표였지만, 여유가 안돼서 구글링을 하다보니 구글에서 NaturalLanguage 를 만들어놓은게 있었다. 바로 안드로 적용해보기ㅎㅎ



### Natural Language API 공식 문서

[GoogleCloud_NaturalLanguage API](https://cloud.google.com/natural-language/docs?hl=ko) 


---



먼저, Cloud Natural Language API를 사용 설정하고 GoogleCloud에서 개인키를 json으로 발급받는다.

(발급받은 json키는 잊어버리지 않도록 조심해야함)

https://console.cloud.google.com/apis/credentials/serviceaccountkey?hl=ko&_ga=2.155617528.2128836955.1602133535-792539781.1598864254&project=esoteric-mote-291314&folder&organizationId



![image-20201008212203515](https://user-images.githubusercontent.com/53978090/95460590-1f5cc680-09b0-11eb-8130-f86bdb452b80.png)


새로운 안드로이드 프로젝트를 생성하고 아까 발급받은 json파일을 layout/raw 안에 넣는다.

raw폴더는 새로 만들어주면된다. 

![image-20201008212411436](https://user-images.githubusercontent.com/53978090/95460600-21bf2080-09b0-11eb-91a0-62dead0ef647.png)


그리고 build.gradle(Module:app) 파일에 다음 코드를 추가해준다.

~~~
ext {
    supportLibraryVersion = '25.3.1'
    googleApiClientVersion = '1.22.0'
}

dependencies {

    implementation fileTree(dir: "libs", include: ["*.jar"])
    implementation 'androidx.appcompat:appcompat:1.1.0'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'androidx.test.ext:junit:1.1.1'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'

// Dependencies for Google API Client Libraries
    implementation("com.google.http-client:google-http-client:$googleApiClientVersion") {
        exclude module: 'httpclient'
        exclude module: 'jsr305'
    }
    implementation("com.google.api-client:google-api-client-android:$googleApiClientVersion"){
        exclude module: 'httpclient'
        exclude module: 'jsr305'
    }
    implementation("com.google.apis:google-api-services-language:v1-rev386-$googleApiClientVersion") {
        exclude module: 'httpclient'
        exclude module: 'jsr305'
    }
}

task copySecretKey(type: Copy) {
    def File secretKey = file "$System.env.GOOGLE_APPLICATION_CREDENTIALS"
    from secretKey.getParent()
    include secretKey.getName()
    into 'src/main/res/raw'
    rename secretKey.getName(), "credential.json"
}
preBuild.dependsOn(copySecretKey)

~~~



sync now 를 해주면 API를 사용할 준비가 다 된거다.

그다음 ! AccessTokenLoader.java를 작성해준다.



**AccessTokenLoader.java**

간단하게 설명하자면 아까 발급받은 json 파일을 불러와서 접근권한 키를 생성해주는 코드라고 보면 된다. 

~~~
package org.techtown.nlptest;

import android.content.Context;
import android.content.SharedPreferences;
import android.util.Log;

import androidx.loader.content.AsyncTaskLoader;

import com.google.api.client.googleapis.auth.oauth2.GoogleCredential;
import com.google.api.services.language.v1.CloudNaturalLanguageScopes;

import java.io.IOException;
import java.io.InputStream;

public class AccessTokenLoader extends AsyncTaskLoader<String> {

    private static final String TAG = "AccessTokenLoader";

    private static final String PREFS = "AccessTokenLoader";
    private static final String PREF_ACCESS_TOKEN = "access_token";

    public AccessTokenLoader(Context context) {
        super(context);
    }

    @Override
    protected void onStartLoading() {
        forceLoad();
    }

    @Override
    public String loadInBackground() {

        final SharedPreferences prefs =
                getContext().getSharedPreferences(PREFS, Context.MODE_PRIVATE);
        String currentToken = prefs.getString(PREF_ACCESS_TOKEN, null);

        // Check if the current token is still valid for a while
        if (currentToken != null) {
            final GoogleCredential credential = new GoogleCredential()
                    .setAccessToken(currentToken)
                    .createScoped(CloudNaturalLanguageScopes.all());
            final Long seconds = credential.getExpiresInSeconds();
            if (seconds != null && seconds > 3600) {
                return currentToken;
            }
        }

        // ***** WARNING *****
        // In this sample, we load the credential from a JSON file stored in a raw resource folder
        // of this client app. You should never do this in your app. Instead, store the file in your
        // server and obtain an access token from there.
        // *******************
        final InputStream stream = getContext().getResources().openRawResource(R.raw.credential);
        try {
            final GoogleCredential credential = GoogleCredential.fromStream(stream)
                    .createScoped(CloudNaturalLanguageScopes.all());
            credential.refreshToken();
            final String accessToken = credential.getAccessToken();
            prefs.edit().putString(PREF_ACCESS_TOKEN, accessToken).apply();
            return accessToken;
        } catch (IOException e) {
            Log.e(TAG, "Failed to obtain access token.", e);
        }
        return null;
    }

}
~~~



분석할 텍스트를 입력하고 분석한 결과를 띄워주는 간단한 뷰를 생성해보자!



**activity_main.xml**

~~~
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <androidx.appcompat.widget.AppCompatEditText
        android:id="@+id/text_et"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginVertical="20dp"
        android:layout_marginHorizontal="10dp"
        android:layout_gravity="center"
        android:hint="분석할 내용을 입력해주세요" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_marginBottom="20dp"
        android:onClick="startAnalysis"
        android:text="start_analysis" />

    <androidx.core.widget.NestedScrollView
        android:id="@+id/nsv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginHorizontal="10dp"
        android:layout_gravity="center"
        android:visibility="gone"
        tools:visibility="visible"
        >

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:layout_margin="8dp"
            android:padding="4dp"
            >

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="result_will_be_displayed_below"
                android:layout_marginBottom="8dp"
                />


            <TextView
                android:id="@+id/result_tv"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="result"

                />

        </LinearLayout>


    </androidx.core.widget.NestedScrollView>


</LinearLayout>
~~~

![image-20201008212857462](https://user-images.githubusercontent.com/53978090/95460604-2388e400-09b0-11eb-8e35-213e5d393d80.png)

**MainActivity.java**

~~~
package org.techtown.nlptest;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.widget.NestedScrollView;
import androidx.loader.app.LoaderManager;
import androidx.loader.content.Loader;

import android.os.Bundle;
import android.text.TextUtils;
import android.util.Log;
import android.view.View;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

import com.google.api.client.googleapis.auth.oauth2.GoogleCredential;
import com.google.api.client.http.HttpRequest;
import com.google.api.client.http.HttpRequestInitializer;
import com.google.api.client.http.javanet.NetHttpTransport;
import com.google.api.client.json.GenericJson;
import com.google.api.client.json.jackson2.JacksonFactory;
import com.google.api.services.language.v1.CloudNaturalLanguage;
import com.google.api.services.language.v1.CloudNaturalLanguageRequest;
import com.google.api.services.language.v1.CloudNaturalLanguageScopes;
import com.google.api.services.language.v1.model.AnalyzeSentimentRequest;
import com.google.api.services.language.v1.model.AnnotateTextRequest;
import com.google.api.services.language.v1.model.Document;
import com.google.api.services.language.v1.model.Features;

import java.io.IOException;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class MainActivity extends AppCompatActivity {
    EditText editTextView; // View to get the text to be analyzed
    TextView resultTextView; // View to display the results obtained after analysis
    NestedScrollView nestedScrollView; // Wrapper view so that if the results are getting out of bounds from screen so the user can scroll and see complete results
    private static final int LOADER_ACCESS_TOKEN = 1; // Token used to initiate the request loader
    private GoogleCredential mCredential = null; //GoogleCredential object so that the requests for NLP Api could be made

    // A  Thread on which the Api request will be made and results will be delivered. As network calls cannot be made on the amin thread, so we are creating a separate thread for the network calls
    private Thread mThread;

    // Google Request for the NLP Api. This actually is acting like a Http client queue that will process each request and response from the Google Cloud server
    private final BlockingQueue<CloudNaturalLanguageRequest<? extends GenericJson>> mRequests
            = new ArrayBlockingQueue<>(3);


    // Api for CloudNaturalLanguage from the Google Client library, this is the instance of our request that we will make to analyze the text.
    private CloudNaturalLanguage mApi = new CloudNaturalLanguage.Builder(
            new NetHttpTransport(),
            JacksonFactory.getDefaultInstance(),
            new HttpRequestInitializer() {
                @Override
                public void initialize(HttpRequest request) throws IOException {
                    mCredential.initialize(request);
                }
            }).build();


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        editTextView = (EditText) findViewById(R.id.text_et);
        resultTextView = (TextView) findViewById(R.id.result_tv);
        nestedScrollView = (NestedScrollView) findViewById(R.id.nsv);
        prepareApi();
    }

    /**
     * Method called on the click of the Button
     * @param view -> the view which is clicked
     */
    public void startAnalysis(View view) {
        String textToAnalyze = editTextView.getText().toString();
        if (TextUtils.isEmpty(textToAnalyze)) {
            editTextView.setError("empty_text_error_msg");
        } else {
            editTextView.setError(null);
            analyzeSentiment(textToAnalyze);
        }
    }

    /**
     * This function will send the text to Cloud Api for analysis
     * @param text -> String to be analyzed
     */
    public void analyzeSentiment(String text) {
        try {
            //blockqueue?에 추가해주기
            mRequests.add(mApi
                    .documents()
                    .analyzeSentiment(new AnalyzeSentimentRequest()
                            .setDocument(new Document()
                                    .setContent(text)
                                    .setType("PLAIN_TEXT"))));
        } catch (IOException e) {
            Log.e("tag", "Failed to create analyze request.", e);
        }
    }

    /**
     * Preparing the Cloud Api before maiking the actual request.
     * This method will actually initiate the AccessTokenLoader async task on completion
     * of which we will recieve the token that should be set in our request for Cloud NLP Api.
     */
    private void prepareApi() {
        // Initiate token refresh
        getSupportLoaderManager().initLoader(LOADER_ACCESS_TOKEN, null,
                new LoaderManager.LoaderCallbacks<String>() {
                    @Override
                    public Loader<String> onCreateLoader(int id, Bundle args) {
                        return new AccessTokenLoader(MainActivity.this);
                    }

                    @Override
                    public void onLoadFinished(Loader<String> loader, String token) {
                        setAccessToken(token);
                    }

                    @Override
                    public void onLoaderReset(Loader<String> loader) {
                    }
                });
    }


    /**
     * This method will set the token from the Credentials.json file to the Google credential object.
     * @param token -> token recieved from the Credentials.json file.
     */
    public void setAccessToken(String token) {
        mCredential = new GoogleCredential()
                .setAccessToken(token)
                .createScoped(CloudNaturalLanguageScopes.all());
        startWorkerThread();
    }


    /**
     * This method will actually initiate a Thread and on this thread we will execute our Api request
     * and responses.
     *
     * Responses recieved will be delivered from here.
     */
    private void startWorkerThread() {
        if (mThread != null) {
            return;
        }
        mThread = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    if (mThread == null) {
                        break;
                    }
                    try {
                        // API calls are executed here in this worker thread
                        deliverResponse(mRequests.take().execute());
                    } catch (InterruptedException e) {
                        Log.e("TAG", "Interrupted.", e);
                        break;
                    } catch (IOException e) {
                        Log.e("TAG", "Failed to execute a request.", e);
                    }
                }
            }
        });
        mThread.start();
    }


    /**
     * this method will handle the response recieved from the Cloud NLP request.
     * The response is a JSON object only.
     * This has been casted to GenericJson from Google Cloud so that the developers can easily parse through the same and can understand the response.
     *
     *
     * @param response --> the JSON object recieved as a response for the cloud NLP Api request
     */
    private void deliverResponse(final GenericJson response) {
        Log.d("TAG", "Generic Response --> " + response);
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Toast.makeText(MainActivity.this, "Response Recieved from Cloud NLP API", Toast.LENGTH_SHORT).show();
                try {
                    resultTextView.setText(response.toPrettyString());
                    nestedScrollView.setVisibility(View.VISIBLE);
                } catch (IOException e) {
                    e.printStackTrace();
                }


            }
        });


    }
}
~~~



나는 감정분석 결과가 필요해서 analyzeSentiment를 사용했지만 NaturalLanguage API에서 제공하는 구문분석, 항목 분석, 콘텐츠 분류등 다양한 기능들도 물론 사용할 수 있다  :thumbsup: :thumbsup:

~~~
    public void analyzeSentiment(String text) {
        try {
            //blockqueue?에 추가해주기
            mRequests.add(mApi
                    .documents()
                    .analyzeSentiment(new AnalyzeSentimentRequest()
                            .setDocument(new Document()
                                    .setContent(text)
                                    .setType("PLAIN_TEXT"))));
        } catch (IOException e) {
            Log.e("tag", "Failed to create analyze request.", e);
        }
    }

~~~

![image-20201008213317889](https://user-images.githubusercontent.com/53978090/95460612-24ba1100-09b0-11eb-8d72-0f15e302f789.png)


**실행 결과**

![image-20201008213403465](https://user-images.githubusercontent.com/53978090/95460617-25eb3e00-09b0-11eb-94fc-5150e46c4ef7.png)

![image-20201008213440123](https://user-images.githubusercontent.com/53978090/95460628-271c6b00-09b0-11eb-9f72-ead2e8ce7315.png)


https://cloud.google.com/natural-language/docs/basics#interpreting_sentiment_analysis_values 

 
![image-20201008213725751](https://user-images.githubusercontent.com/53978090/95460633-284d9800-09b0-11eb-9533-bcb452814d32.png)


감정 분석 값 해석을 살펴보면 -1.0(부정) ~ 1.0(긍정) 의 범위로 score가 나타나는걸 볼 수 있다. 대체로 점수값이 해당 범위내로 나오는거 같긴하다! 하지만 한글이라 그런지 일기처럼 여러문장으로 사용하면 값이 애매하게 나오는거같기도 함..ㅎ 



---

### 참조

https://mobikul.com/integrating-cloud-natural-language-api-in-android/

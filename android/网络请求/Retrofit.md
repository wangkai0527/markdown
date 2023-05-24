[TOC]

# Retrofit

首先说明以下为什么要把这3个东西放在一起，其实主要是想介绍 Retrofit ，当时网上很多文章上来就是各种用法，对 我们刚刚接触框架的人不太友好，所以想写这篇文章来展示一下它们之间到底有什么关联。

Retrofit 我们知道是一个现在非常流行的网络请求库，它流行的关键在于它非常好使用，对Http请求做了封装，让我们使用起来更加方便，并且还提供了基于Anotation的 Rest风格的请求方式，如果使用过SpringMVC的人就能感觉到，这两者结合起来，可以后台和客户端开发契合得更好。
OkHttp 是 Retrofit 底层使用的Http请求库，都是 Square 公司的开源产品。OkHttp 和 Android中的 ，HttpUriConneciton才是一级的产品，而Retrofit 底层使用作为Http通信的工具的就是 OkHttp。
RxJava 其实和 Retrofit 并没有什么关系，但是由于它流式编程的思想，丰富的操作符，线程的任意切换等优点广受大家的喜爱。特别是用在像做网络请求这样比较繁琐的逻辑处理中，更能显示出它的威力，所以将RxJava和Retrofit结合起来，威力无比强大。
但是我也刚刚接触，学习总结以下，所以这篇文章也只针对新手而已，大神请自行绕过。


参考网站
OkHttp官网  Okio官网  Retrofit官网

OkHttp部分
要使用OkHttp，首先需要添加它的依赖，同时OkHttp又依赖了Okio，所以需要同时添加Okio的依赖。

OkHttp依赖
maven: 
<dependency>
  <groupId>com.squareup.okhttp3</groupId>
  <artifactId>okhttp</artifactId>
  <version>3.6.0</version>
</dependency>

gradle: 

    compile 'com.squareup.okhttp3:okhttp:3.6.0'


Okio依赖
maven: 
<dependency>
    <groupId>com.squareup.okio</groupId>
    <artifactId>okio</artifactId>
    <version>1.11.0</version>
</dependency>

gradle: 

compile 'com.squareup.okio:okio:1.11.0'


使用OkHttp下载图片例子 
```Java
class DownloadImageThread extends Thread {
    private String url;
    private ImageView imageView;

    public DownloadImageThread(String url, ImageView imageView) {
      this.url = url;
      this.imageView = imageView;
    }

    @Override
    public void run() {
      OkHttpClient client = new OkHttpClient();
      Request request = new Request.Builder()
          .url(url)
          .build();

      try {
        Response response = client.newCall(request).execute();
        if (response.isSuccessful()) {
          final Bitmap bitmap = BitmapFactory.decodeStream(response.body().byteStream());
          runOnUiThread(new Runnable() {
            @Override
            public void run() {
              imageView.setImageBitmap(bitmap);
            }
          });
        }
      } catch (IOException e) {
        e.printStackTrace();
      }
    }
  }
```

Retrofit部分

Retrofit依赖
Maven： 
<dependency>
  <groupId>com.squareup.retrofit2</groupId>
  <artifactId>retrofit</artifactId>
  <version>2.1.0</version>
</dependency>

Gradle： 

    compile 'com.squareup.retrofit2:retrofit:2.1.0'


Gson依赖 
    
    compile 'com.google.code.gson:gson:2.3.1'


返回数据转换器依赖
在Retrofit中将转换器称为 Converters，它的作用就是用来添加对返回数据的转换支持。例如的是String 就需要 scalar，如果是 json，需要使用 gson 等。后面的代码中可以看到

Scalars (primitives, boxed, and String): com.squareup.retrofit2:converter-scalars:2.1.0
Gson: com.squareup.retrofit2:converter-gson:2.1.0
Jackson: com.squareup.retrofit2:converter-jackson:{最新版本号}
Moshi: com.squareup.retrofit2:converter-moshi:{最新版本号}
Protobuf: com.squareup.retrofit2:converter-protobuf:{最新版本号}
Wire: com.squareup.retrofit2:converter-wire:{最新版本号}
Simple XML: com.squareup.retrofit2:converter-simplexml:{最新版本号}


例子所有依赖
接下来两个例子将展示怎么使用Retrofit请求Github api的数据。 

dependencies {
    //...
    compile 'com.squareup.retrofit2:converter-scalars:2.1.0'
    compile 'com.squareup.retrofit2:converter-gson:2.1.0'
    compile 'com.squareup.retrofit2:retrofit:2.1.0'
    compile 'com.google.code.gson:gson:2.3.1'
}

                            
请求String数据例子
GitHubService.Java 定义发出Http请求的接口。两个接口，一个返回String，一个返回GitModel（Github返回的Json数据的Pojo类）

public interface GitHubService {

  String BASEURL = "https://api.github.com";

  @GET("users/{user}")
  Call<String> getData(@Path("user") String user);

  @GET("users/{user}")
  Call<GitModel> getUserInfo(@Path("user") String user);
}

GitModel.java 注意：这里省略了 getters 和 setters，还需要 Gson 支持 

public class GitModel {
  @Expose
  private String login;
  @Expose
  private Integer id;
  @SerializedName("avatar_url")
  @Expose
  private String avatarUrl;
  @SerializedName("gravatar_id")
  @Expose
  private String gravatarId;
  @Expose
  private String url;
  @SerializedName("html_url")
  @Expose
  private String htmlUrl;
  @SerializedName("followers_url")
  @Expose
  private String followersUrl;
  @SerializedName("following_url")
  @Expose
  private String followingUrl;
  @SerializedName("gists_url")
  @Expose
  private String gistsUrl;
  @SerializedName("starred_url")
  @Expose
  private String starredUrl;
  @SerializedName("subscriptions_url")
  @Expose
  private String subscriptionsUrl;
  @SerializedName("organizations_url")
  @Expose
  private String organizationsUrl;
  @SerializedName("repos_url")
  @Expose
  private String reposUrl;
  @SerializedName("events_url")
  @Expose
  private String eventsUrl;
  @SerializedName("received_events_url")
  @Expose
  private String receivedEventsUrl;
  @Expose
  private String type;
  @SerializedName("site_admin")
  @Expose
  private Boolean siteAdmin;
  @Expose
  private String name;
  @Expose
  private String company;
  @Expose
  private String blog;
  @Expose
  private String location;
  @Expose
  private String email;
  @Expose
  private Boolean hireable;
  @Expose
  private Object bio;
  @SerializedName("public_repos")
  @Expose
  private Integer publicRepos;
  @SerializedName("public_gists")
  @Expose
  private Integer publicGists;
  @Expose
  private Integer followers;
  @Expose
  private Integer following;
  @SerializedName("created_at")
  @Expose
  private String createdAt;
  @SerializedName("updated_at")
  @Expose
  private String updatedAt;

  // Getters
  // ...

  // Setters
  // ...
}

MainActivity.java 

Retrofit retrofit = new Retrofit.Builder()
    .baseUrl(GitHubService.BASEURL)
    //添加String支持
    .addConverterFactory(ScalarsConverterFactory.create())
    .build();

GitHubService service = retrofit.create(GitHubService.class);
Call<String> call = service.getData(username);//username 可以自己传入github的用户名

// 异步请求
call.enqueue(new Callback<String>() {
  @Override
  public void onResponse(Call<String> call, Response<String> response) {
    // 处理返回数据
    if (response.isSuccessful()) {
      Log.d(TAG, "onResponse: " + response.body());
    }
  }
  @Override
  public void onFailure(Call<String> call, Throwable t) {
    Log.d(TAG, "onFailure: 请求数据失败");
  }
});

                            
请求Json数据例子
如果是Json数据，使用Gson自动解析

Retrofit retrofit = new Retrofit.Builder()
    .baseUrl(GitHubService.BASEURL)
    // 添加Json转换器支持
    .addConverterFactory(GsonConverterFactory.create())
    .build();
GitHubService service = retrofit.create(GitHubService.class);
Call<GitModel> call = service.getUserInfo(username);//username 可填入自己Github账号用户名
call.enqueue(new Callback<GitModel>() {
  @Override
  public void onResponse(Call<GitModel> call, Response<GitModel> response) {
    if (response.isSuccessful()) {
      Log.d(TAG, "onResponse: " + response.body().getName());
    }
  }
  @Override
  public void onFailure(Call<GitModel> call, Throwable t) {
    Log.d(TAG, "onFailure: " + t.getMessage());
  }
});

                            
使用 ResponseBody
当然，我们还可以直接使用 ResponseBody，还记得前面我们介绍过OkHttp，其实Retrofit 底层使用的 也是OkHttp，所以我们的请求完全也可以使用 OkHttp 的 Response 对象 中的 ResponseBody 对象来处理返回数据。
GitHubService.java 

public interface GitHubService {
  String BASEURL = "https://api.github.com";

  //...

  @GET("users/{user}")
  Call<ResponseBody> getResponseBody(@Path("user") String user);
}

获取 ResponseBody，然后处理方法就和 OkHttp 一样了 

GitHubService service = retrofit.create(GitHubService.class);
Call<ResponseBody> call = service.getResponseBody(username);
call.enqueue(new Callback<ResponseBody>() {
  @Override
  public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response) {
    if(response.isSuccessful()){
      try {
        Log.d(TAG, "onResponse: "+response.body().string());
      } catch (IOException e) {
        e.printStackTrace();
      }
    }
  }

  @Override
  public void onFailure(Call<ResponseBody> call, Throwable t) {
    Log.d(TAG, "onFailure: ");
  }
});

                            
使用RxJava
要使用RxJava和Retrofit的结合，首先我们要添加要给依赖 

compile 'io.reactivex:rxjava:1.1.7'
compile 'io.reactivex:rxandroid:1.2.1'
compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'

再添加一个新的访问数据方法 

public interface GitHubService {
  String BASEURL = "https://api.github.com";

  //...
  @GET("users/{user}")
  Observable<GitModel> rxUser(@Path("user") String user);
}

使用Rxjava访问 

public void rxRetrofit(String username) {
  Retrofit retrofit = new Retrofit.Builder()
      .baseUrl(GitHubService.BASEURL)
      .addConverterFactory(GsonConverterFactory.create())
      .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
      .build();

  GitHubService service = retrofit.create(GitHubService.class);
  Observable<GitModel> obserable = service.rxUser(username);
  obserable
      .subscribeOn(Schedulers.io())
      .observeOn(AndroidSchedulers.mainThread())
      .subscribe(new Subscriber<GitModel>() {
        @Override
        public void onCompleted() {
          Log.d(TAG, "onCompleted: ");
        }

        @Override
        public void onError(Throwable e) {
          Log.d(TAG, "onError: " + e.getMessage());
        }

        @Override
        public void onNext(GitModel gitModel) {
          Log.d(TAG, "onNext: " + gitModel.getName());
        }
      });
}

在使用时发现代码不但没有减少，而且增多了，其实不然。这只是展示的一个最基本的例子，RxJava的强大之处在于它流式编程的思想，强大的操作符，以及线程之间的切换。当我们有比较复杂的逻辑的时候，它的强大就显现出来了，这里只是为了演示用法。








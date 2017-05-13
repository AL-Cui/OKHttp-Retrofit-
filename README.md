# OKHttp-Retrofit-
OkHttp和Retrofit的入门级知识

一、OkHttp初识
=============

<p>Android系统提供了两种HTTP通信类，HttpURLConnection和HttpClient。
关于HttpURLConnection和HttpClient的选择>>官方博客
尽管Google在大部分安卓版本中推荐使用HttpURLConnection，但是这个类相比HttpClient实在是太难用，太弱爆了。
OkHttp是一个相对成熟的解决方案，据说Android4.4的源码中可以看到HttpURLConnection已经替换成OkHttp实现了。OkHttp好在哪里呢？支持连接同一地址的链接共享同一个socket，通过连接池来减小响应延迟，还有透明的GZIP压缩，请求缓存等优势，其核心主要有路由、连接协议、拦截器、代理、安全性认证、连接池以
及网络适配，拦截器主要是指添加，移除或者转换请求或者回应的头部信息。第一缺点是消息回来需要切到主线程，主线程要自己去写，第二传入调用比较复杂。</p>
---------------

二、Retrofit初识
=========

<p>Retrofit是Square公司开源的一个高质量高效率的http库.它将我们自己开发的底层的代码和细节都封装了起来,只是暴露出了我们业务中的数据模型和操作方法。</p>

Retrofit的特性
-------------

<p>将rest API封装为java接口，我们根据业务需求来进行接口的封装，实际开发可能会封装多个不同的java接口以满足业务需求。这时候你需要了解Retrofit的注解，关于注解请参考

[此博客](http://blog.csdn.net/qiang_xi/article/details/53959437)
使用Retrofit提供的封装方法生成我们接口的实现类，这个真的很赞，不用我们自己实现，通过注解Retrofit全部帮我们自动生成好了。
</p>
三、下面我们来写一个简单的调用（GET、POST）
=====================

第一步，导入所依赖的包
------------
<p>在你的Android studio中找到build.gradle文件，添加如下代码</p>

    compile "com.squareup.retrofit2:retrofit:$retrofitVersion"
    
    compile "com.squareup.retrofit2:converter-gson:$retrofitVersion"


第二步，将Rest API转换为java接口
--------------
    public interface GitHubService {
    
    @GET("/users/{username}")
    
    Call<User> getUser(@Path("username") String username);
    
    @Headers({"Content-type:application/json;charset=UTF-8"})
    
    @POST("device/aircondition/get?access_token=YmlndWl5dWFuLXNoYXJwLWFpcnB1cmlmaWVy")
    
    Call<AirInfo> getAirInfo(
            @Body RequestBody jsonBean);
            
      }
      
<p>在上面的代码中我们定义了一个POST请求和一个GET请求</p>

第三步，Retrofit会帮我们自动生成接口的实现类的实例
-----------

<p>我们首先实现GET请求的</p>

```
 public class ServiceGenerator {
    
    public static final String API_BASE_URL = "https://api.github.com";

    private static OkHttpClient.Builder httpClient = new OkHttpClient.Builder();

    private static Retrofit.Builder builder =
            new Retrofit.Builder()
                    .baseUrl(API_BASE_URL)
                    .addConverterFactory(GsonConverterFactory.create());

    public static <S> S createService(Class<S> serviceClass) {
        return createService(serviceClass, null);
    }

    public static <S> S createService(Class<S> serviceClass, final String authToken) {
        if (authToken != null) {
            httpClient.interceptors().add(new Interceptor() {
                @Override
                public okhttp3.Response intercept(Interceptor.Chain chain) throws IOException {
                    Request original = chain.request();

                    // Request customization: add request headers
                    Request.Builder requestBuilder = original.newBuilder()
                            .header("Authorization", authToken)
                            .method(original.method(), original.body());

                    Request request = requestBuilder.build();
                    return chain.proceed(request);
                }
            });
        }

        OkHttpClient client = httpClient.build();
        Retrofit retrofit = builder.client(client).build();
        return retrofit.create(serviceClass);   
    }    
 }
```
第四步，建立我们的解析model
==========

```Java
public class GitHubModel {
    private GitHubService git;
    private MainViewModel viewModel;


    public GitHubModel(MainViewModel viewModel) {
        this.viewModel = viewModel;
        this.git =  ServiceGenerator.createService(GitHubService.class);

    }

   public void getUser(String username) {
        //binding.username.getText().toString()
        Call call = git.getUser(username);
        call.enqueue(new Callback<User>() {
            @Override
            public void onResponse(Response<User> response) {
                User model = response.body();

                if (model == null) {
                    //404 or the response cannot be converted to User.
                    ResponseBody responseBody = response.errorBody();
                    if (responseBody != null) {
                        try {
                            viewModel.setText("responseBody = " + responseBody.string());
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    } else {
                        viewModel.setText("responseBody  = null");
                    }
                } else {
                    //200
                    viewModel.setText("Github Name :" + model.getName() + "\nWebsite :" + model.getBlog() + "\nCompany Name :" + model.getCompany());
                }
                viewModel.setPb(false);
            }

            @Override
            public void onFailure(Throwable t) {
                viewModel.setText("t = " + t.getMessage());
                viewModel.setPb(false);
            }
        });
    }

}
```
<p>在解析model中我们用到了User Bean文件，用来存放从服务器中获取的数据</p>
```Java
public class User {

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
    
}

省略Getter和Setter方法
```



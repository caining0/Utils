# Utils
常用工具类
上面不再解释
下面写一下对okhttputils的封装请求


/**
 * Created by oslanka on 2015/10/20.
 */
public class BaseEngine<T> {
    public static BaseEngine baseEngine;

    public static BaseEngine getInstance() {
        // if (null == baseEngine) {
        boolean connect = NetWorkState.getInstance().isConnect(BaseApp.getAppContext());
        if (!connect){
            ToastUtils.show(BaseApp.getAppContext(),"当前网络不可用，请检查你的网络设置！");
        }
        baseEngine = new BaseEngine();
        // }
        return baseEngine;
    }

    public void getTPost(final String url, final Map<String, String> parms, final Type type, final CallbackForT<T> callback) {
        OkHttpUtils.post().url(url).params(parms)
                .headers(getHeader())
                .build().execute(new Callback() {
            @Override
            public Object parseNetworkResponse(Response response) throws IOException {
                return response.body().string();
            }

            @Override
            public void onError(Request request, Exception e) {
            }

            @Override
            public void onResponse(Object response) {
                LogUtils.Log(response.toString());
                paserjson(response.toString(), callback, type);

            }
        });
    }

    public void getTGet(final String url, final Map<String, String> parms, final Type type, final CallbackForT<T> callback) {
        OkHttpUtils.get().url(url).params(parms)
                .headers(getHeader())
                .build().execute(new Callback() {
            @Override
            public Object parseNetworkResponse(Response response) throws IOException {
                return response.body().string();
            }

            @Override
            public void onError(Request request, Exception e) {
            }

            @Override
            public void onResponse(Object response) {
                LogUtils.Log(response.toString());
                paserjson(response.toString(), callback, type);
            }
        });
    }

    /**
     * json 解析
     * @param json
     * @param callback
     * @param type
     */

    private void paserjson(String json, CallbackForT<T> callback, Type type) {
        JSONObject jsonObject =null;
        try {
            jsonObject = new JSONObject(json);
        } catch (JSONException e) {
            e.printStackTrace();
        }

        if (TextUtils.isEmpty(json)||TextUtils.isEmpty(jsonObject.optString("data"))||jsonObject.optString("data")==null)
        {
            ToastUtils.show(BaseApp.getAppContext(),"没有更多数据");
            return;
        }
        Gson gson = new Gson();
        try {
            if (json.trim().startsWith("[")) {
                List<T> retList = gson.fromJson(json, type);
                if (retList!=null)
                callback.finish(retList);
            } else if (json.trim().startsWith("{")){
                T beans = gson.fromJson(json, type);
                if (beans!=null)
                callback.finish(beans);
            }
        } catch (IllegalStateException e) {

        }
    }


    public void getStringPost(final String url, final Map<String, String> parms, final CallbackForT<String> callback) {

        OkHttpUtils.post().url(url).params(parms)
                .headers(getHeader())
                .build().execute(new Callback() {
            @Override
            public Object parseNetworkResponse(Response response) throws IOException {
                return response.body().string();
            }

            @Override
            public void onError(Request request, Exception e) {

            }

            @Override
            public void onResponse(Object response) {
                LogUtils.Log(response.toString());
                callback.finish(response.toString());
            }
        });
    }

    public void getStringGet(final String url, final Map<String, String> parms, final CallbackForT<String> callback) {
        OkHttpUtils.get().url(url).params(parms)
                .headers(getHeader())
                .build()
                .execute(new Callback() {
                    @Override
                    public Object parseNetworkResponse(Response response) throws IOException {
                        return response.body().string();
                    }

                    @Override
                    public void onError(Request request, Exception e) {
                        LogUtils.Log(e.toString());
                    }

                    @Override
                    public void onResponse(Object response) {
                        LogUtils.Log(response.toString());
                        callback.finish(response.toString());
                    }
                });
    }

    public void setFilePost(final String url, File file,final Map<String, String> parms, final CallbackForT<String> callback) {
        Map<String, String> headers = getHeader();
        headers.put("Content-Type", "image/jpeg");
        RequestCall call = null;
        if (file!=null) {
            call = OkHttpUtils.post()//
                    .addFile("file", "CropImage.jpeg", file)
                    .url(url)
                    .params(parms)
                    .headers(headers)
                    .build();
        }else {
            call = OkHttpUtils.post()//
                    .url(url)
                    .params(parms)
                    .headers(headers)
                    .build();
        }
        call.execute(new Callback() {
            @Override
            public Object parseNetworkResponse(Response response) throws IOException {
                return response.body().string();
            }

            @Override
            public void onError(Request request, Exception e) {
                LogUtils.Log(e.toString());
            }

            @Override
            public void onResponse(Object response) {
                callback.finish(response.toString());
            }
        });
    }
    public void getResponse(String url, final CallbackForResponse callback){
        OkHttpUtils.get().url(url).build().execute(new Callback() {
            @Override
            public Object parseNetworkResponse(Response response) throws IOException {
                return response;
            }

            @Override
            public void onError(Request request, Exception e) {

            }

            @Override
            public void onResponse(Object response) {
                callback.finish((byte[])response);
            }
        });
    }



    public interface CallbackForT<T> {
        void finish(T bean);

        void finish(List<T> listT);
    }
    public interface  CallbackForResponse{
        void finish(byte[] response);
    }


//    ------------------------------------------------------------------------------------------------------------------------
//    ------------------------------------------------------------------------------------------------------------------------
//    ------------------------------------------------------------------------------------------------------------------------
//    ------------------------------------------------------------------------------------------------------------------------
//    ------------------------------------------------------------------------------------------------------------------------

    private Map<String, String> getHeader() {
        Map<String, String> headers = new HashMap<>();
        headers.put("Content-Type", "application/x-app.H5Plugin.www-form-urlencoded; charset=utf-8");
        String strBase64 = Base64.encodeToString(PhoneInfo.getInstance().getPhoneInfo(BaseApp.getAppContext()).getBytes(), Base64.NO_WRAP);
        headers.put("OLENV", strBase64);
        headers.put("Authorization", "Basic " + Base64.encodeToString((PhoneInfo.openId + ":" + PhoneInfo.username).getBytes(), Base64.NO_WRAP));
        if (PhoneInfo.getInstance().isApkDebugable(BaseApp.getAppContext()))//de
        {
            LogUtils.Log(Configs.CONNCTTO170);
            headers.put("TESTENV", Configs.CONNCTTO170);//1为170
        }
        return headers;
    }
}

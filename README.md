mainactivity 代码如下：public class MainActivity extends AppCompatActivity {
    private TextView tvCity;
    private TextView tvTemp;
    private TextView tvWind;
    private TextView tvHum;
    private TextView tvUpdtime;
    private ListView listWeather;

    private Handler handler=new Handler() {

        @Override
        public void handleMessage(@NonNull Message msg) {
            super.handleMessage(msg);
            String obj = (String) msg.obj;
            parseJson(obj);
            //下半部分内容
            Bundle data = msg.getData();
            WeatherBean weather = (WeatherBean) data.getSerializable("wheather");
            setListview(weather);

        }
    };

        private void setListview(WeatherBean weather) {
            List<Map<String,String>> data=new ArrayList<>();
            List<WeatherBean.ResultBean.DailyBean> daily=weather.getResult().getDaily();
            for(int i=0;i<daily.size();i++){
                Map<String,String>map=new HashMap<>();
                map.put("date",daily.get(i).getDate());
                map.put("week",daily.get(i).getWeek());
                map.put("imglow",daily.get(i).getNight().getImg()+".png");
                map.put("templow",daily.get(i).getNight().getTemplow());
                map.put("imghigh",daily.get(i).getDay().getImg()+".png");
                map.put("temphigh",daily.get(i).getDay().getTemphigh());
                data.add(map);

            }


            SimpleAdapter adapter=new SimpleAdapter(MainActivity.this,data,R.layout.weather_list_item_layout,
            new String[]{"date","week","imglow","imghigh","templow","temphigh"},
                    new int[]{R.id.tv_date,R.id.tv_week,R.id.img_low,R.id.img_high,R.id.tv_low,R.id.tv_high});
            adapter.setViewBinder(new SimpleAdapter.ViewBinder() {
                @Override
                public boolean setViewValue(View view, Object data, String textRepresentation) {
                    if(view instanceof ImageView){
                        try {
                            InputStream in=getResources().getAssets().open(data.toString());
                            Bitmap bitmap=BitmapFactory.decodeStream(in);
                            ((ImageView) view).setImageBitmap(bitmap);
                            return true;
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                    return false;
                }
            });
            listWeather.setAdapter(adapter);
        }


    private void parseJson(String obj) {
        try {
            JSONObject object=new JSONObject(obj);
            String result=object.optString("result");
            object=new JSONObject(result);
            String city= object.getString("city");
            String temphigh=object.getString("temphigh");
            String winddirect=object.getString("winddirect");
            String windpower=object.getString("windpower");
            String humidity=object.getString("humidity");

            tvCity.setText(city);
            tvTemp.setText(temphigh+"℃");
            tvWind.setText(winddirect+windpower);
            tvHum.setText("湿度："+humidity+"%");

        }catch (JSONException e){
            e.printStackTrace();
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_weather);
        getNetData();

        initViews();
    }

    private void initViews() {

        tvCity = (TextView) findViewById(R.id.tv_city);
        tvTemp = (TextView) findViewById(R.id.tv_temp);
        tvWind = (TextView) findViewById(R.id.tv_wind);
        tvHum = (TextView) findViewById(R.id.tv_hum);
        tvUpdtime = (TextView) findViewById(R.id.tv_updtime);
        listWeather = (ListView) findViewById(R.id.list_weather);

    }

    private void getNetData() {//访问网络获得天气数据
        new Thread(){
            @Override
            public void run() {
                super.run();
                OkHttpClient client=new OkHttpClient();
                Request.Builder builder=new Request.Builder();
                builder.url("https://api.jisuapi.com/weather/query?appkey=a09e5727081a1c8c&city=安顺");
                Request request=builder.build();
                Call call=client.newCall(request);
                try {
                    Response response=call.execute();
                    if(response.isSuccessful()){
                        String result=response.body().string();
                        Gson gson=new Gson();
                        WeatherBean weatherBean=gson.fromJson(result,WeatherBean.class);

                        Message msg=handler.obtainMessage();
                        msg.obj=result;
                        Bundle bundle=new Bundle();
                        bundle.putSerializable("wheather",weatherBean);
                        msg.setData(bundle);
                        handler.sendMessage(msg);//
                        Log.i("okhttp","run"+result);
                    }
                }catch (IOException e){
                    e.printStackTrace();
                }

            }
        }.start();

    }
}
WeatherBean代码如下：package com.example.demo0529;

import java.io.Serializable;
import java.util.List;

public class WeatherBean implements Serializable {

    /**
     * status : 0
     * msg : ok
     * result : {"city":"安顺","cityid":111,"citycode":"101260301","date":"2023-05-29","week":"星期一","temp":"21","temphigh":"28","templow":"19","humidity":"84","pressure":"847","windspeed":"8.9","winddirect":"南风","windpower":"3级","updatetime":"2023-05-29 09:20:00","index":[{"iname":"空调指数","ivalue":"较少开启","detail":"您将感到很舒适，一般不需要开启空调。"},{"iname":"运动指数","ivalue":"较适宜","detail":"天气较好，户外运动请注意防晒。推荐您进行室内运动。"},{"iname":"紫外线指数","ivalue":"强","detail":"紫外线辐射强，建议涂擦SPF20左右、PA++的防晒护肤品。避免在10点至14点暴露于日光下。"},{"iname":"感冒指数","ivalue":"少发","detail":"各项气象条件适宜，无明显降温过程，发生感冒机率较低。"},{"iname":"洗车指数","ivalue":"较适宜","detail":"较适宜洗车，未来一天无雨，风力较小，擦洗一新的汽车至少能保持一天。"},{"iname":"空气污染扩散指数","ivalue":"较差","detail":"气象条件较不利于空气污染物稀释、扩散和清除，请适当减少室外活动时间。"},{"iname":"穿衣指数","ivalue":"热","detail":"天气热，建议着短裙、短裤、短薄外套、T恤等夏季服装。"}],"aqi":{"so2":"5","so224":"","no2":"4","no224":"","co":"0.4","co24":"","o3":"64","o38":"","o324":"","pm10":"21","pm1024":"","pm2_5":"17","pm2_524":"","iso2":"","ino2":"","ico":"","io3":"","io38":"","ipm10":"","ipm2_5":"","aqi":"25","primarypollutant":"PM25","quality":"优","timepoint":"2023-05-29 08:00:00","aqiinfo":{"level":"一级","color":"#00e400","affect":"空气质量令人满意，基本无空气污染","measure":"各类人群可正常活动"}},"daily":[{"date":"2023-05-29","week":"星期一","sunrise":"06:05","sunset":"19:42","night":{"weather":"多云","templow":"19","img":"1","winddirect":"南风","windpower":"微风"},"day":{"weather":"晴","temphigh":"28","img":"0","winddirect":"南风","windpower":"微风"}},{"date":"2023-05-30","week":"星期二","sunrise":"06:05","sunset":"19:43","night":{"weather":"阵雨","templow":"20","img":"3","winddirect":"东风","windpower":"微风"},"day":{"weather":"阵雨","temphigh":"27","img":"3","winddirect":"东北风","windpower":"微风"}},{"date":"2023-05-31","week":"星期三","sunrise":"06:04","sunset":"19:43","night":{"weather":"阵雨","templow":"20","img":"3","winddirect":"南风","windpower":"微风"},"day":{"weather":"多云","temphigh":"27","img":"1","winddirect":"东南风","windpower":"微风"}},{"date":"2023-06-01","week":"星期四","sunrise":"06:04","sunset":"19:44","night":{"weather":"阵雨","templow":"19","img":"3","winddirect":"东南风","windpower":"微风"},"day":{"weather":"阵雨","temphigh":"27","img":"3","winddirect":"南风","windpower":"微风"}},{"date":"2023-06-02","week":"星期五","sunrise":"06:04","sunset":"19:44","night":{"weather":"阵雨","templow":"18","img":"3","winddirect":"南风","windpower":"微风"},"day":{"weather":"阵雨","temphigh":"26","img":"3","winddirect":"南风","windpower":"微风"}},{"date":"2023-06-03","week":"星期六","sunrise":"06:04","sunset":"19:45","night":{"weather":"阵雨","templow":"19","img":"3","winddirect":"南风","windpower":"微风"},"day":{"weather":"阵雨","temphigh":"25","img":"3","winddirect":"南风","windpower":"微风"}},{"date":"2023-06-04","week":"星期日","sunrise":"06:04","sunset":"19:45","night":{"weather":"小雨","templow":"16","img":"7","winddirect":"北风","windpower":"微风"},"day":{"weather":"中雨","temphigh":"25","img":"8","winddirect":"南风","windpower":"微风"}}],"hourly":[{"time":"8:00","weather":"小雨","temp":"20","img":7},{"time":"9:00","weather":"阴","temp":"22","img":2},{"time":"10:00","weather":"阴","temp":"23","img":2},{"time":"11:00","weather":"阴","temp":"25","img":2},{"time":"12:00","weather":"阴","temp":"26","img":2},{"time":"13:00","weather":"阴","temp":"27","img":2},{"time":"14:00","weather":"多云","temp":"28","img":1},{"time":"15:00","weather":"多云","temp":"28","img":1},{"time":"16:00","weather":"多云","temp":"28","img":1},{"time":"17:00","weather":"晴","temp":"28","img":0},{"time":"18:00","weather":"晴","temp":"27","img":0},{"time":"19:00","weather":"晴","temp":"25","img":0},{"time":"20:00","weather":"晴","temp":"24","img":0},{"time":"21:00","weather":"晴","temp":"24","img":0},{"time":"22:00","weather":"晴","temp":"23","img":0},{"time":"23:00","weather":"晴","temp":"22","img":0},{"time":"0:00","weather":"晴","temp":"22","img":0},{"time":"1:00","weather":"晴","temp":"21","img":0},{"time":"2:00","weather":"晴","temp":"20","img":0},{"time":"3:00","weather":"晴","temp":"20","img":0},{"time":"4:00","weather":"晴","temp":"19","img":0},{"time":"5:00","weather":"晴","temp":"19","img":0},{"time":"6:00","weather":"晴","temp":"20","img":0},{"time":"7:00","weather":"晴","temp":"21","img":0}],"weather":"阴","img":"2"}
     */

    private int status;
    private String msg;
    private ResultBean result;

    public int getStatus() {
        return status;
    }

    public void setStatus(int status) {
        this.status = status;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public ResultBean getResult() {
        return result;
    }

    public void setResult(ResultBean result) {
        this.result = result;
    }

    public static class ResultBean {
        /**
         * city : 安顺
         * cityid : 111
         * citycode : 101260301
         * date : 2023-05-29
         * week : 星期一
         * temp : 21
         * temphigh : 28
         * templow : 19
         * humidity : 84
         * pressure : 847
         * windspeed : 8.9
         * winddirect : 南风
         * windpower : 3级
         * updatetime : 2023-05-29 09:20:00
         * index : [{"iname":"空调指数","ivalue":"较少开启","detail":"您将感到很舒适，一般不需要开启空调。"},{"iname":"运动指数","ivalue":"较适宜","detail":"天气较好，户外运动请注意防晒。推荐您进行室内运动。"},{"iname":"紫外线指数","ivalue":"强","detail":"紫外线辐射强，建议涂擦SPF20左右、PA++的防晒护肤品。避免在10点至14点暴露于日光下。"},{"iname":"感冒指数","ivalue":"少发","detail":"各项气象条件适宜，无明显降温过程，发生感冒机率较低。"},{"iname":"洗车指数","ivalue":"较适宜","detail":"较适宜洗车，未来一天无雨，风力较小，擦洗一新的汽车至少能保持一天。"},{"iname":"空气污染扩散指数","ivalue":"较差","detail":"气象条件较不利于空气污染物稀释、扩散和清除，请适当减少室外活动时间。"},{"iname":"穿衣指数","ivalue":"热","detail":"天气热，建议着短裙、短裤、短薄外套、T恤等夏季服装。"}]
         * aqi : {"so2":"5","so224":"","no2":"4","no224":"","co":"0.4","co24":"","o3":"64","o38":"","o324":"","pm10":"21","pm1024":"","pm2_5":"17","pm2_524":"","iso2":"","ino2":"","ico":"","io3":"","io38":"","ipm10":"","ipm2_5":"","aqi":"25","primarypollutant":"PM25","quality":"优","timepoint":"2023-05-29 08:00:00","aqiinfo":{"level":"一级","color":"#00e400","affect":"空气质量令人满意，基本无空气污染","measure":"各类人群可正常活动"}}
         * daily : [{"date":"2023-05-29","week":"星期一","sunrise":"06:05","sunset":"19:42","night":{"weather":"多云","templow":"19","img":"1","winddirect":"南风","windpower":"微风"},"day":{"weather":"晴","temphigh":"28","img":"0","winddirect":"南风","windpower":"微风"}},{"date":"2023-05-30","week":"星期二","sunrise":"06:05","sunset":"19:43","night":{"weather":"阵雨","templow":"20","img":"3","winddirect":"东风","windpower":"微风"},"day":{"weather":"阵雨","temphigh":"27","img":"3","winddirect":"东北风","windpower":"微风"}},{"date":"2023-05-31","week":"星期三","sunrise":"06:04","sunset":"19:43","night":{"weather":"阵雨","templow":"20","img":"3","winddirect":"南风","windpower":"微风"},"day":{"weather":"多云","temphigh":"27","img":"1","winddirect":"东南风","windpower":"微风"}},{"date":"2023-06-01","week":"星期四","sunrise":"06:04","sunset":"19:44","night":{"weather":"阵雨","templow":"19","img":"3","winddirect":"东南风","windpower":"微风"},"day":{"weather":"阵雨","temphigh":"27","img":"3","winddirect":"南风","windpower":"微风"}},{"date":"2023-06-02","week":"星期五","sunrise":"06:04","sunset":"19:44","night":{"weather":"阵雨","templow":"18","img":"3","winddirect":"南风","windpower":"微风"},"day":{"weather":"阵雨","temphigh":"26","img":"3","winddirect":"南风","windpower":"微风"}},{"date":"2023-06-03","week":"星期六","sunrise":"06:04","sunset":"19:45","night":{"weather":"阵雨","templow":"19","img":"3","winddirect":"南风","windpower":"微风"},"day":{"weather":"阵雨","temphigh":"25","img":"3","winddirect":"南风","windpower":"微风"}},{"date":"2023-06-04","week":"星期日","sunrise":"06:04","sunset":"19:45","night":{"weather":"小雨","templow":"16","img":"7","winddirect":"北风","windpower":"微风"},"day":{"weather":"中雨","temphigh":"25","img":"8","winddirect":"南风","windpower":"微风"}}]
         * hourly : [{"time":"8:00","weather":"小雨","temp":"20","img":7},{"time":"9:00","weather":"阴","temp":"22","img":2},{"time":"10:00","weather":"阴","temp":"23","img":2},{"time":"11:00","weather":"阴","temp":"25","img":2},{"time":"12:00","weather":"阴","temp":"26","img":2},{"time":"13:00","weather":"阴","temp":"27","img":2},{"time":"14:00","weather":"多云","temp":"28","img":1},{"time":"15:00","weather":"多云","temp":"28","img":1},{"time":"16:00","weather":"多云","temp":"28","img":1},{"time":"17:00","weather":"晴","temp":"28","img":0},{"time":"18:00","weather":"晴","temp":"27","img":0},{"time":"19:00","weather":"晴","temp":"25","img":0},{"time":"20:00","weather":"晴","temp":"24","img":0},{"time":"21:00","weather":"晴","temp":"24","img":0},{"time":"22:00","weather":"晴","temp":"23","img":0},{"time":"23:00","weather":"晴","temp":"22","img":0},{"time":"0:00","weather":"晴","temp":"22","img":0},{"time":"1:00","weather":"晴","temp":"21","img":0},{"time":"2:00","weather":"晴","temp":"20","img":0},{"time":"3:00","weather":"晴","temp":"20","img":0},{"time":"4:00","weather":"晴","temp":"19","img":0},{"time":"5:00","weather":"晴","temp":"19","img":0},{"time":"6:00","weather":"晴","temp":"20","img":0},{"time":"7:00","weather":"晴","temp":"21","img":0}]
         * weather : 阴
         * img : 2
         */

        private String city;
        private int cityid;
        private String citycode;
        private String date;
        private String week;
        private String temp;
        private String temphigh;
        private String templow;
        private String humidity;
        private String pressure;
        private String windspeed;
        private String winddirect;
        private String windpower;
        private String updatetime;
        private AqiBean aqi;
        private String weather;
        private String img;
        private List<IndexBean> index;
        private List<DailyBean> daily;
        private List<HourlyBean> hourly;

        public String getCity() {
            return city;
        }

        public void setCity(String city) {
            this.city = city;
        }

        public int getCityid() {
            return cityid;
        }

        public void setCityid(int cityid) {
            this.cityid = cityid;
        }

        public String getCitycode() {
            return citycode;
        }

        public void setCitycode(String citycode) {
            this.citycode = citycode;
        }

        public String getDate() {
            return date;
        }

        public void setDate(String date) {
            this.date = date;
        }

        public String getWeek() {
            return week;
        }

        public void setWeek(String week) {
            this.week = week;
        }

        public String getTemp() {
            return temp;
        }

        public void setTemp(String temp) {
            this.temp = temp;
        }

        public String getTemphigh() {
            return temphigh;
        }

        public void setTemphigh(String temphigh) {
            this.temphigh = temphigh;
        }

        public String getTemplow() {
            return templow;
        }

        public void setTemplow(String templow) {
            this.templow = templow;
        }

        public String getHumidity() {
            return humidity;
        }

        public void setHumidity(String humidity) {
            this.humidity = humidity;
        }

        public String getPressure() {
            return pressure;
        }

        public void setPressure(String pressure) {
            this.pressure = pressure;
        }

        public String getWindspeed() {
            return windspeed;
        }

        public void setWindspeed(String windspeed) {
            this.windspeed = windspeed;
        }

        public String getWinddirect() {
            return winddirect;
        }

        public void setWinddirect(String winddirect) {
            this.winddirect = winddirect;
        }

        public String getWindpower() {
            return windpower;
        }

        public void setWindpower(String windpower) {
            this.windpower = windpower;
        }

        public String getUpdatetime() {
            return updatetime;
        }

        public void setUpdatetime(String updatetime) {
            this.updatetime = updatetime;
        }

        public AqiBean getAqi() {
            return aqi;
        }

        public void setAqi(AqiBean aqi) {
            this.aqi = aqi;
        }

        public String getWeather() {
            return weather;
        }

        public void setWeather(String weather) {
            this.weather = weather;
        }

        public String getImg() {
            return img;
        }

        public void setImg(String img) {
            this.img = img;
        }

        public List<IndexBean> getIndex() {
            return index;
        }

        public void setIndex(List<IndexBean> index) {
            this.index = index;
        }

        public List<DailyBean> getDaily() {
            return daily;
        }

        public void setDaily(List<DailyBean> daily) {
            this.daily = daily;
        }

        public List<HourlyBean> getHourly() {
            return hourly;
        }

        public void setHourly(List<HourlyBean> hourly) {
            this.hourly = hourly;
        }

        public static class AqiBean {
            /**
             * so2 : 5
             * so224 :
             * no2 : 4
             * no224 :
             * co : 0.4
             * co24 :
             * o3 : 64
             * o38 :
             * o324 :
             * pm10 : 21
             * pm1024 :
             * pm2_5 : 17
             * pm2_524 :
             * iso2 :
             * ino2 :
             * ico :
             * io3 :
             * io38 :
             * ipm10 :
             * ipm2_5 :
             * aqi : 25
             * primarypollutant : PM25
             * quality : 优
             * timepoint : 2023-05-29 08:00:00
             * aqiinfo : {"level":"一级","color":"#00e400","affect":"空气质量令人满意，基本无空气污染","measure":"各类人群可正常活动"}
             */

            private String so2;
            private String so224;
            private String no2;
            private String no224;
            private String co;
            private String co24;
            private String o3;
            private String o38;
            private String o324;
            private String pm10;
            private String pm1024;
            private String pm2_5;
            private String pm2_524;
            private String iso2;
            private String ino2;
            private String ico;
            private String io3;
            private String io38;
            private String ipm10;
            private String ipm2_5;
            private String aqi;
            private String primarypollutant;
            private String quality;
            private String timepoint;
            private AqiinfoBean aqiinfo;

            public String getSo2() {
                return so2;
            }

            public void setSo2(String so2) {
                this.so2 = so2;
            }

            public String getSo224() {
                return so224;
            }

            public void setSo224(String so224) {
                this.so224 = so224;
            }

            public String getNo2() {
                return no2;
            }

            public void setNo2(String no2) {
                this.no2 = no2;
            }

            public String getNo224() {
                return no224;
            }

            public void setNo224(String no224) {
                this.no224 = no224;
            }

            public String getCo() {
                return co;
            }

            public void setCo(String co) {
                this.co = co;
            }

            public String getCo24() {
                return co24;
            }

            public void setCo24(String co24) {
                this.co24 = co24;
            }

            public String getO3() {
                return o3;
            }

            public void setO3(String o3) {
                this.o3 = o3;
            }

            public String getO38() {
                return o38;
            }

            public void setO38(String o38) {
                this.o38 = o38;
            }

            public String getO324() {
                return o324;
            }

            public void setO324(String o324) {
                this.o324 = o324;
            }

            public String getPm10() {
                return pm10;
            }

            public void setPm10(String pm10) {
                this.pm10 = pm10;
            }

            public String getPm1024() {
                return pm1024;
            }

            public void setPm1024(String pm1024) {
                this.pm1024 = pm1024;
            }

            public String getPm2_5() {
                return pm2_5;
            }

            public void setPm2_5(String pm2_5) {
                this.pm2_5 = pm2_5;
            }

            public String getPm2_524() {
                return pm2_524;
            }

            public void setPm2_524(String pm2_524) {
                this.pm2_524 = pm2_524;
            }

            public String getIso2() {
                return iso2;
            }

            public void setIso2(String iso2) {
                this.iso2 = iso2;
            }

            public String getIno2() {
                return ino2;
            }

            public void setIno2(String ino2) {
                this.ino2 = ino2;
            }

            public String getIco() {
                return ico;
            }

            public void setIco(String ico) {
                this.ico = ico;
            }

            public String getIo3() {
                return io3;
            }

            public void setIo3(String io3) {
                this.io3 = io3;
            }

            public String getIo38() {
                return io38;
            }

            public void setIo38(String io38) {
                this.io38 = io38;
            }

            public String getIpm10() {
                return ipm10;
            }

            public void setIpm10(String ipm10) {
                this.ipm10 = ipm10;
            }

            public String getIpm2_5() {
                return ipm2_5;
            }

            public void setIpm2_5(String ipm2_5) {
                this.ipm2_5 = ipm2_5;
            }

            public String getAqi() {
                return aqi;
            }

            public void setAqi(String aqi) {
                this.aqi = aqi;
            }

            public String getPrimarypollutant() {
                return primarypollutant;
            }

            public void setPrimarypollutant(String primarypollutant) {
                this.primarypollutant = primarypollutant;
            }

            public String getQuality() {
                return quality;
            }

            public void setQuality(String quality) {
                this.quality = quality;
            }

            public String getTimepoint() {
                return timepoint;
            }

            public void setTimepoint(String timepoint) {
                this.timepoint = timepoint;
            }

            public AqiinfoBean getAqiinfo() {
                return aqiinfo;
            }

            public void setAqiinfo(AqiinfoBean aqiinfo) {
                this.aqiinfo = aqiinfo;
            }

            public static class AqiinfoBean {
                /**
                 * level : 一级
                 * color : #00e400
                 * affect : 空气质量令人满意，基本无空气污染
                 * measure : 各类人群可正常活动
                 */

                private String level;
                private String color;
                private String affect;
                private String measure;

                public String getLevel() {
                    return level;
                }

                public void setLevel(String level) {
                    this.level = level;
                }

                public String getColor() {
                    return color;
                }

                public void setColor(String color) {
                    this.color = color;
                }

                public String getAffect() {
                    return affect;
                }

                public void setAffect(String affect) {
                    this.affect = affect;
                }

                public String getMeasure() {
                    return measure;
                }

                public void setMeasure(String measure) {
                    this.measure = measure;
                }
            }
        }

        public static class IndexBean {
            /**
             * iname : 空调指数
             * ivalue : 较少开启
             * detail : 您将感到很舒适，一般不需要开启空调。
             */

            private String iname;
            private String ivalue;
            private String detail;

            public String getIname() {
                return iname;
            }

            public void setIname(String iname) {
                this.iname = iname;
            }

            public String getIvalue() {
                return ivalue;
            }

            public void setIvalue(String ivalue) {
                this.ivalue = ivalue;
            }

            public String getDetail() {
                return detail;
            }

            public void setDetail(String detail) {
                this.detail = detail;
            }
        }

        public static class DailyBean {
            /**
             * date : 2023-05-29
             * week : 星期一
             * sunrise : 06:05
             * sunset : 19:42
             * night : {"weather":"多云","templow":"19","img":"1","winddirect":"南风","windpower":"微风"}
             * day : {"weather":"晴","temphigh":"28","img":"0","winddirect":"南风","windpower":"微风"}
             */

            private String date;
            private String week;
            private String sunrise;
            private String sunset;
            private NightBean night;
            private DayBean day;

            public String getDate() {
                return date;
            }

            public void setDate(String date) {
                this.date = date;
            }

            public String getWeek() {
                return week;
            }

            public void setWeek(String week) {
                this.week = week;
            }

            public String getSunrise() {
                return sunrise;
            }

            public void setSunrise(String sunrise) {
                this.sunrise = sunrise;
            }

            public String getSunset() {
                return sunset;
            }

            public void setSunset(String sunset) {
                this.sunset = sunset;
            }

            public NightBean getNight() {
                return night;
            }

            public void setNight(NightBean night) {
                this.night = night;
            }

            public DayBean getDay() {
                return day;
            }

            public void setDay(DayBean day) {
                this.day = day;
            }

            public static class NightBean {
                /**
                 * weather : 多云
                 * templow : 19
                 * img : 1
                 * winddirect : 南风
                 * windpower : 微风
                 */

                private String weather;
                private String templow;
                private String img;
                private String winddirect;
                private String windpower;

                public String getWeather() {
                    return weather;
                }

                public void setWeather(String weather) {
                    this.weather = weather;
                }

                public String getTemplow() {
                    return templow;
                }

                public void setTemplow(String templow) {
                    this.templow = templow;
                }

                public String getImg() {
                    return img;
                }

                public void setImg(String img) {
                    this.img = img;
                }

                public String getWinddirect() {
                    return winddirect;
                }

                public void setWinddirect(String winddirect) {
                    this.winddirect = winddirect;
                }

                public String getWindpower() {
                    return windpower;
                }

                public void setWindpower(String windpower) {
                    this.windpower = windpower;
                }
            }

            public static class DayBean {
                /**
                 * weather : 晴
                 * temphigh : 28
                 * img : 0
                 * winddirect : 南风
                 * windpower : 微风
                 */

                private String weather;
                private String temphigh;
                private String img;
                private String winddirect;
                private String windpower;

                public String getWeather() {
                    return weather;
                }

                public void setWeather(String weather) {
                    this.weather = weather;
                }

                public String getTemphigh() {
                    return temphigh;
                }

                public void setTemphigh(String temphigh) {
                    this.temphigh = temphigh;
                }

                public String getImg() {
                    return img;
                }

                public void setImg(String img) {
                    this.img = img;
                }

                public String getWinddirect() {
                    return winddirect;
                }

                public void setWinddirect(String winddirect) {
                    this.winddirect = winddirect;
                }

                public String getWindpower() {
                    return windpower;
                }

                public void setWindpower(String windpower) {
                    this.windpower = windpower;
                }
            }
        }

        public static class HourlyBean {
            /**
             * time : 8:00
             * weather : 小雨
             * temp : 20
             * img : 7
             */

            private String time;
            private String weather;
            private String temp;
            private int img;

            public String getTime() {
                return time;
            }

            public void setTime(String time) {
                this.time = time;
            }

            public String getWeather() {
                return weather;
            }

            public void setWeather(String weather) {
                this.weather = weather;
            }

            public String getTemp() {
                return temp;
            }

            public void setTemp(String temp) {
                this.temp = temp;
            }

            public int getImg() {
                return img;
            }

            public void setImg(int img) {
                this.img = img;
            }
        }
    }
}
activity_weather代码如下：<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView android:id="@+id/tv_city"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="20sp"
        android:textColor="#00f"
        android:text="城市名" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
        <TextView android:id="@+id/tv_temp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="30sp"
            android:textColor="#00f"
            android:text="温度" />
        <TextView android:id="@+id/tv_wind"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="15sp"
            android:textColor="#00f"
            android:text="风力风向" />
    </LinearLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
        <TextView android:id="@+id/tv_hum"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="30sp"
            android:textColor="#00f"
            android:text="湿度" />
        <TextView android:id="@+id/tv_updtime"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="15sp"
            android:textColor="#00f"
            android:text="更新时间" />
    </LinearLayout>
    <ListView android:id="@+id/list_weather"
        android:layout_weight="1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"></ListView>
</LinearLayout> 
weather_list_item_layout代码如下：<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <TextView android:layout_weight="1"
        android:id="@+id/tv_date"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="日期" />
    <TextView android:layout_weight="1"
        android:id="@+id/tv_week"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="周几" />

    <LinearLayout android:layout_weight="1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal">


        <ImageView
            android:id="@+id/img_low"
            android:layout_width="30dp"
            android:layout_height="30dp"
            android:src="@mipmap/ic_launcher" />

        <ImageView
            android:id="@+id/img_high"
            android:layout_width="30dp"
            android:layout_height="30dp"
            android:src="@mipmap/ic_launcher" />
    </LinearLayout>

    <LinearLayout android:layout_weight="1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <TextView
            android:id="@+id/tv_low"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="低温" />
        <TextView

            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="/" />
        <TextView
            android:id="@+id/tv_high"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="高温" />
    </LinearLayout>

</LinearLayout>

完善代码满足下面要求“【任务描述】网络上有一些天气预报API接口，能够获得相关城市的天气信息。例如：
https://api.jisuapi.com/weather/query?appkey=a09e5727081a1c8c&city=安顺
能返回安顺市六天的天气预报信息。
该网站数据来源: 国家气象局www.weather.com.cn
更新频率: 每小时更新一次
 
1.请根据用户设定的城市，进行网络查询，显示该城市的天气信息，包括实时天气情况、最高最低温度、风级、风力等指数，以及该城市最近6天的天气情况，主界面如右图所示。
2.能进行城市切换。
3.能够记忆上次选择的城市，下次启动时能够自动显示该城市的天气信息。
”

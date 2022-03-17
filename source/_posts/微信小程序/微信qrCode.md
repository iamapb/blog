---
title: 生成对应小程序对应路径的二维码
date: 2019-03-04 18:16:31
tags: wx
---
### java 生成小程序某路径的二维码
```java

            CloseableHttpClient httpClient = HttpClients.createDefault();
            HttpGet httpGet = new HttpGet("https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=" + 你的appId+ "&secret=" + Constant.WX_APP_SECRET);
            CloseableHttpResponse resp;
            String accessToken = "";
            try {
                resp = httpClient.execute(httpGet);
                HttpEntity entity = resp.getEntity();
                String result = EntityUtils.toString(entity, "UTF-8");
                HashMap hashMap = objectMapper.readValue(result, HashMap.class);
                accessToken = (String) hashMap.get("access_token");
            } catch (IOException e) {
                e.printStackTrace();
            }
            CourseModel courseModel = this.getById(id);
            String path = "你的微信小程序路径";
            String url = "https://api.weixin.qq.com/wxa/getwxacodeunlimit?access_token=" + accessToken;
            HttpPost httpPost = new HttpPost(url);
    
            Map<String, String> map = new HashMap<String, String>();
            map.put("scene", id.toString());
            map.put("page", "pages"+path+"main");
           
            String jsonEncode = Util.jsonEncode(map);
            StringEntity stringEntity = null;
            try {
                 stringEntity = new StringEntity(jsonEncode, Charset.forName("UTF-8"));
            } catch (Exception e) {
                e.printStackTrace();
            }
            httpPost.setEntity(stringEntity);
            httpPost.addHeader("Content-type", "application/json; charset=utf-8");
            httpPost.setHeader("Accept", "application/json");
            CloseableHttpResponse response1 = null;
            try {
                response1 = httpClient.execute(httpPost);
            } catch (IOException e) {
                e.printStackTrace();
            }
            InputStream content = null;
            try {
                content = response1.getEntity().getContent();
            } catch (IOException e) {
                e.printStackTrace();
            }
    //        response.addHeader("Content-Type", "image/jpeg");
            try {
                IOUtils.copy(content, response.getOutputStream());
            } catch (IOException e) {
                e.printStackTrace();
            }

```

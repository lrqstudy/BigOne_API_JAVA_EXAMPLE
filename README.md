# BigOne_API_JAVA_EXAMPLE

package com.libra.exchange.exchanges.bigone;

import com.google.api.client.util.Maps;
import org.json.JSONObject;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.*;
import java.math.BigDecimal;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;
import java.util.Map;
import java.util.UUID;

/**
 * Created by rongqiang.li on 18/2/15.
 * <p>
 * contact: lrqstudy@gmail.com
 */
public class BigOneExample {
    public final static String BIGONE_BASE_URL = "https://api.big.one";

    public final static String BIGONE_GET_ACCOUNT_COMMAND = "/accounts";
    public final static String BIGONE_ORDERS_COMMAND = "/orders";


    private static final String USER_AGENT = "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36";

    private static Logger logger = LoggerFactory.getLogger(BigOneExample.class);

    public static final String BIG_ONE_API_KEY = "YourApiKey";// https://big.one/settings

    public static void main(String[] args) {
        testGetAccountInfoList();
        testTrade();
    }

    private static void testGetAccountInfoList() {
        BigOneExample bigOneExample = new BigOneExample();
        bigOneExample.getAccountInfoList();
    }

    private static void testTrade() {
        //0.00102BTC 价格购买 10EOS
        BigOneExample bigOneExample = new BigOneExample();
        BigDecimal price = new BigDecimal("0.00106");
        BigDecimal amount = new BigDecimal("10");
        amount = amount.setScale(2, BigDecimal.ROUND_CEILING);
        bigOneExample.trade("EOS-BTC", price, amount, 2, "ASK");//buy
    }

    public void getOpenOrderList(String currencyPair) {
        StringBuilder sb = new StringBuilder();
        sb.append(BIGONE_BASE_URL).append(BIGONE_ORDERS_COMMAND).append("?market=").append(currencyPair);

        String result = executeCommandWithGet(sb.toString());
        System.out.println(result);
    }


    public void getAccountInfoList() {
        try {
            StringBuilder sb = new StringBuilder();
            sb.append(BIGONE_BASE_URL).
                    append(BIGONE_GET_ACCOUNT_COMMAND).
                    append("/");
            String resultStr = executeCommandWithGet(sb.toString());
            System.out.println(resultStr);
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
        }
    }


    public String buy(String currrencyPair, BigDecimal price, BigDecimal amount, Integer precisionSize) {

        amount = amount.setScale(precisionSize, BigDecimal.ROUND_CEILING);
        String result = trade(currrencyPair, price, amount, precisionSize, "BID");

        return result;
    }

    private String trade(String currencyPair, BigDecimal price, BigDecimal amount, Integer precisionSize, String orderSide) {
        try {
            Map<String, String> params = Maps.newHashMap();
            amount = amount.setScale(precisionSize, BigDecimal.ROUND_CEILING);//设置最小的交易量精度,如果是 10.01 这样的格式精度设置为2, 如果为6.3256 这样的精度设置为4,表示小数后四位
            params.put("order_market", currencyPair);
            params.put("order_side", orderSide);
            params.put("price", price.toPlainString());
            params.put("amount", amount.toPlainString());

            StringBuilder sb = new StringBuilder();
            sb.append(BIGONE_BASE_URL).append(BIGONE_ORDERS_COMMAND);

            String result = executeCommandWithPost(sb.toString(), params);
            System.out.println(result);
            return result;
        } catch (Exception e) {
            logger.error(" sell error" + e.getMessage(), e);
            return "error";
        }
    }


    public String sell(String currrencyPair, BigDecimal price, BigDecimal amount, Integer precisionSize) {
        Map<String, String> params = Maps.newHashMap();
        amount = amount.setScale(precisionSize, BigDecimal.ROUND_CEILING);
        params.put("order_market", currrencyPair);
        params.put("order_side", "ASK");
        params.put("price", price.toPlainString());
        params.put("amount", amount.toPlainString());
        return trade(currrencyPair, price, amount, precisionSize, "ASK");
    }


    private String executeCommandWithPost(String urlStr, Map<String, String> paramsMap) {
        HttpURLConnection conn = null;
        try {
            URL url = new URL(urlStr);
            conn = (HttpURLConnection) url.openConnection();
            conn.setUseCaches(false);
            conn.setDoOutput(true);
            conn.setRequestProperty("Authorization", buildAuthorizationHeaderValue());
            conn.setRequestProperty("Big-Device-Id", UUID.randomUUID().toString());
            conn.setRequestProperty("Content-Type", "application/json");
            conn.setRequestProperty("User-Agent", USER_AGENT);
            conn.setRequestMethod("POST");


            String postData;
            JSONObject jsonObject = new JSONObject();
            for (Map.Entry<String, String> entry : paramsMap.entrySet()) {
                jsonObject.put(entry.getKey(), entry.getValue());
            }

            postData = jsonObject.toString();
            OutputStreamWriter out = new OutputStreamWriter(conn.getOutputStream());

            out.write(postData);
            out.close();
            String responseResult = convertStreamToString(conn.getInputStream());
            logger.info(" call url={}, responseResult={} ", responseResult);
            return responseResult;
        } catch (MalformedURLException e) {
            throw new RuntimeException("call url  error" + e.getMessage(), e);
        } catch (Exception e) {
            throw new RuntimeException("call url  error" + e.getMessage(), e);
        } finally {
            if (conn != null) {
                conn.disconnect();
            }
        }
    }

    private String buildAuthorizationHeaderValue() {
        StringBuilder sb = new StringBuilder();
        //TODO you api key in bigone ;Bearer 是固定写法请照抄,这块我当时也没想到是这样的,跟我调试过大部分的 API不一样,以为 Bearer 是个代名词,用各种东西试了都是400,发现还是自己太无知,不知道这是一种特定写法
        sb.append("Bearer ").append(BIG_ONE_API_KEY);//https://big.one/settings
        return sb.toString();
    }


    private String executeCommandWithGet(String urlStr) {
        HttpURLConnection conn = null;
        try {
            URL url = new URL(urlStr);
            conn = (HttpURLConnection) url.openConnection();
            conn.setUseCaches(false);
            conn.setDoOutput(true);
            conn.setRequestProperty("Authorization", buildAuthorizationHeaderValue());
            conn.setRequestProperty("Big-Device-Id", UUID.randomUUID().toString());
            conn.setRequestProperty("Content-Type", "application/json");
            conn.setRequestProperty("User-Agent", USER_AGENT);
            conn.setRequestMethod("GET");
            String responseResult = convertStreamToString(conn.getInputStream());
            logger.info(" call url={}, responseResult={} ", responseResult);
            return responseResult;
        } catch (MalformedURLException e) {
            throw new RuntimeException("call url  error" + e.getMessage(), e);
        } catch (IOException e) {
            throw new RuntimeException("call url  error" + e.getMessage(), e);
        } finally {
            if (conn != null) {
                conn.disconnect();
            }
        }
    }

    private String convertStreamToString(InputStream is) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(is));
        StringBuilder sb = new StringBuilder();

        String line;
        try {
            while ((line = reader.readLine()) != null) {
                sb.append(line).append('\n');
            }
        } finally {
            try {
                is.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return sb.toString();
    }
}

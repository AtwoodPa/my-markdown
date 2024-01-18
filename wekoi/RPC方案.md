## Dubbo
统一使用dubbo作为rpc暴露和调用方式，协议使用dubbo

- 示例
  - 服务提供者
  - 服务消费者

## 协议
- RMI
- Hessian/Burlap
- HttpInvoker
- thrift
- Restful

## 使用
esb(Enterprise Service Bus,企业服务总线)提供RPC统一访问接口
```java
package suishen.weather.api.rpc;

import suishen.esb.meta.RpcResult;
import suishen.weather.common.PageWrapper;
import suishen.weather.meta.horoscope.AdminHoroscopeQueryParam;
import suishen.weather.meta.horoscope.AdminHoroscopeQueryResp;
import suishen.weather.meta.horoscope.AdminHoroscopeUpdateParam;

/**
 * 紫薇星座 - rpc接口
 *
 * @author supanpan
 * @date 2023/10/17
 */
public interface IWeatherHoroscopeService {
    /**
     * 查询 - 紫薇星座数据列表
     *
     * @param param 查询参数
     * @return 返回结果
     */
    RpcResult<PageWrapper<AdminHoroscopeQueryResp>> queryHoroscopeList(AdminHoroscopeQueryParam param);

    /**
     * 更新 - 紫薇星座数据
     *
     * @param param 编辑参数
     * @return 更新结果
     */
    RpcResult<Boolean> updateHoroscope(AdminHoroscopeUpdateParam param);


}

```
生产者
weather
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
http://dubbo.apache.org/schema/dubbo
http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="zhwnl_weather">
        <dubbo:parameter key="qos.enable" value="false"/>
    </dubbo:application>

    <!-- 使用zk注册中心暴露服务地址 -->
    <dubbo:registry address="zookeeper://node1.zk.all.platform.wtc.hwhosts.com:2181?backup=node2.zk.all.platform.wtc.hwhosts.com:2181,node3.zk.all.platform.wtc.hwhosts.com:2181" timeout="10000"/>

    <dubbo:protocol name="dubbo" port="${weather.dubbo.port}" threads="1200"/>


    <!-- Dubbo 消费者 -->
    <dubbo:consumer check="false"/>
    <!-- 提供者 -->
    <dubbo:service interface="suishen.weather.api.rpc.IWeatherHoroscopeService"
                 ref="weatherHoroscopeRpcServiceImpl" protocol="dubbo" timeout="3000" retries="2" loadbalance="random"/>
</beans>
```
RPC提供接口
```java
/**
 * 紫薇星座 - rpc接口
 *
 * @author supanpan
 * @date 2023/10/17
 */
public interface IWeatherHoroscopeService {
    /**
     * 查询 - 紫薇星座数据列表
     *
     * @param param 查询参数
     * @return 返回结果
     */
    RpcResult<PageWrapper<AdminHoroscopeQueryResp>> queryHoroscopeList(AdminHoroscopeQueryParam param);

    /**
     * 更新 - 紫薇星座数据
     *
     * @param param 编辑参数
     * @return 更新结果
     */
    RpcResult<Boolean> updateHoroscope(AdminHoroscopeUpdateParam param);


}
```
weather实现RPC接口
```java
package suishen.rpc.horoscope;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import suishen.esb.meta.RpcResult;
import suishen.libs.util.JSONUtil;
import suishen.libs.web.exception.BusinessException;
import suishen.service.horoscope.service.WeatherHoroscopeService;
import suishen.weather.api.rpc.IWeatherHoroscopeService;
import suishen.weather.common.PageWrapper;
import suishen.weather.meta.horoscope.AdminHoroscopeQueryParam;
import suishen.weather.meta.horoscope.AdminHoroscopeQueryResp;
import suishen.weather.meta.horoscope.AdminHoroscopeUpdateParam;

import javax.annotation.Resource;

/**
 * 紫薇星座数据RPC接口
 *
 * @author supanpan
 * @date 2023/10/17
 */
@Slf4j
@Service("weatherHoroscopeRpcServiceImpl")
public class WeatherHoroscopeServiceImpl implements IWeatherHoroscopeService {
    @Resource
    private WeatherHoroscopeService weatherHoroscopeService;

    /**
     * rpc 查询紫薇星座数据接口
     *
     * @param param 查询参数
     * @return 结果
     */
    @Override
    public RpcResult<PageWrapper<AdminHoroscopeQueryResp>> queryHoroscopeList(AdminHoroscopeQueryParam param) {
        // RPC响应结果集合
        RpcResult<PageWrapper<AdminHoroscopeQueryResp>> rpcResult = new RpcResult<>();
        try{
            rpcResult.setData(weatherHoroscopeService.getHoroscopeListForRpc(param));
            rpcResult.setStatus(RpcResult.ResultStatus.SUCCESS);
            rpcResult.setMsg(RpcResult.ResultStatus.SUCCESS.getDesc());
        }catch (BusinessException e){// 业务异常
            rpcResult.setStatus(RpcResult.ResultStatus.PARAM_ERROR);
            rpcResult.setMsg(e.getMessage());
        }catch (Exception e){// 服务端异常
            log.error("【Horoscope RPC】queryHoroscopeList error", e);
            rpcResult.setStatus(RpcResult.ResultStatus.SERVER_ERROR);
            rpcResult.setMsg(RpcResult.ResultStatus.SERVER_ERROR.getDesc());
        }
        return rpcResult;
    }

    /**
     * rpc 更新紫薇星座数据接口
     * @param param 编辑参数
     * @return
     */
    @Override
    public RpcResult<Boolean> updateHoroscope(AdminHoroscopeUpdateParam param) {
        // RPC响应结果集合
        RpcResult<Boolean> rpcResult = new RpcResult<>();
        try {
            rpcResult.setData(weatherHoroscopeService.editHoroscopeListForRpc(param));
            rpcResult.setStatus(RpcResult.ResultStatus.SUCCESS);
            rpcResult.setMsg(RpcResult.ResultStatus.SUCCESS.getDesc());
        } catch (BusinessException e) {
            rpcResult.setStatus(RpcResult.ResultStatus.PARAM_ERROR);
            rpcResult.setMsg(e.getMessage());
        } catch (Exception e) {
            log.error("【Horoscope RPC】updateHoroscope error, param={}", JSONUtil.getJsonString(param), e);
            rpcResult.setStatus(RpcResult.ResultStatus.SERVER_ERROR);
            rpcResult.setMsg(RpcResult.ResultStatus.SERVER_ERROR.getDesc());
        }
        return rpcResult;
    }
}

```
WeatherService
```java
package suishen.service.horoscope.service;

import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.springframework.stereotype.Service;
import suishen.libs.web.exception.BusinessException;
import suishen.libs.web.meta.ActionStatus;
import suishen.service.horoscope.cache.HoroscopeCache;
import suishen.service.horoscope.dao.HoroscopeAdminDao;
import suishen.service.horoscope.meta.HoroscopeAdminBean;
import suishen.service.horoscope.meta.HoroscopeBean;
import suishen.weather.common.PageWrapper;
import suishen.weather.meta.horoscope.AdminHoroscopeQueryParam;
import suishen.weather.meta.horoscope.AdminHoroscopeQueryResp;
import suishen.weather.meta.horoscope.AdminHoroscopeUpdateParam;

import javax.annotation.Resource;
import java.util.ArrayList;
import java.util.List;

/**
 * @author supanpan
 * @date 2023/10/19
 */
@Slf4j
@Service
public class WeatherHoroscopeService {

    @Resource
    private HoroscopeCache horoscopeCache;


    @Resource
    private HoroscopeService horoscopeService;


    @Resource
    private HoroscopeAdminDao horoscopeAdminDao;

    /**
     * 查询紫薇星座数据列表（rpc）
     *
     * @param param 查询参数
     * @return 结果
     */
    public PageWrapper<AdminHoroscopeQueryResp> getHoroscopeListForRpc(AdminHoroscopeQueryParam param) {
        PageWrapper<AdminHoroscopeQueryResp> pageWrapper = new PageWrapper<>(param.getPageNum(), param.getPageSize());
        // 总记录数
        int count = horoscopeAdminDao.countHoroscopeListByParams(param);
        if (count < 0){
            return pageWrapper;
        }
        pageWrapper.setTotalCount(count);
        List<HoroscopeAdminBean> horoscopeList = horoscopeAdminDao.getHoroscopeListByParams(param);

        pageWrapper.setList(convertData(horoscopeList));
        return pageWrapper;
    }

    /**
     * 更新紫薇星座数据（rpc）
     * @param param 更新参数
     * @return 更新结果
     */
    public Boolean editHoroscopeListForRpc(AdminHoroscopeUpdateParam param) {
        if (param == null
                || param.getId() <= 0
                || StringUtils.isEmpty(param.getAiqingyun())
                || StringUtils.isEmpty(param.getShiyeyun())
                || StringUtils.isEmpty(param.getCaiyun())
                || StringUtils.isEmpty(param.getDescription())){
            throw new BusinessException(ActionStatus.PARAMAS_ERROR);
        }
        // 查询原有数据
        HoroscopeAdminBean horoscopeAdminBean = horoscopeAdminDao.getById(param.getId());
        // 更新数据
        horoscopeAdminBean.setAiqingyun(param.getAiqingyun());
        horoscopeAdminBean.setShiyeyun(param.getShiyeyun());
        horoscopeAdminBean.setCaiyun(param.getCaiyun());
        horoscopeAdminBean.setDescription(param.getDescription());
        boolean result = horoscopeAdminDao.updateHoroscope(param);
        // 更新缓存数据
        if (result){
            // 刷新Redis内数据
            HoroscopeBean horoscope = horoscopeService.getHoroscopeFromDB(horoscopeAdminBean.getType(), horoscopeAdminBean.getDay(), horoscopeAdminBean.getTimetype());
            horoscopeCache.setHoroscope2Redis(horoscope);
            horoscopeCache.invalidate(horoscopeAdminBean.getType(), horoscopeAdminBean.getDay(), horoscopeAdminBean.getTimetype());

        }
        return result;
    }

    /**
     * 格式转换
     *
     * @param horoscopeList
     * @return
     */
    private List<AdminHoroscopeQueryResp> convertData(List<HoroscopeAdminBean> horoscopeList){
        List<AdminHoroscopeQueryResp> resultList = new ArrayList<>();
        for (HoroscopeAdminBean horoscope : horoscopeList){
            try{
                AdminHoroscopeQueryResp result = AdminHoroscopeQueryResp.builder()
                        .id(horoscope.getId())
                        .day(horoscope.getDay())
                        .type(horoscope.getType())
                        .zhonghe(horoscope.getZhonghe())
                        .aiqing(horoscope.getAiqing())
                        .gongzuo(horoscope.getGongzuo())
                        .touzi(horoscope.getTouzi())
                        .jiankang(horoscope.getJiankang())
                        .yanse(horoscope.getYanse())
                        .shuzi(horoscope.getShuzi())
                        .supei(horoscope.getSupei())
                        .description(horoscope.getDescription())
                        .name(horoscope.getName())
                        .aiqingyun(horoscope.getAiqingyun())
                        .shiyeyun(horoscope.getShiyeyun())
                        .caiyun(horoscope.getCaiyun())
                        .timetype(horoscope.getTimetype())
                        .fangwei(horoscope.getFangwei())
                        .yueyoushi(horoscope.getYueyoushi())
                        .yuelueshi(horoscope.getYuelueshi())
                        .xingyunyue(horoscope.getXingyunyue())
                        .build();

                resultList.add(result);
            }catch (Exception e) {
                log.error(e.getMessage(),e);
            }
        }
        return resultList;
    }
}

```
Cache
```java
package suishen.service.horoscope.cache;

import com.google.common.cache.Cache;
import com.google.common.collect.Lists;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.collections.CollectionUtils;
import org.apache.commons.lang3.StringUtils;
import org.springframework.stereotype.Component;
import suishen.common.Constant;
import suishen.common.RedisKey;
import suishen.libs.util.GuavaCache;
import suishen.libs.util.JSONUtil;
import suishen.redis.SuishenRedisTemplate;
import suishen.service.horoscope.dao.HoroscopeDao;
import suishen.service.horoscope.dto.HoroscopeDayInfoDTO;
import suishen.service.horoscope.enums.HoroscopeTimeTypeEnum;
import suishen.service.horoscope.meta.HoroscopeAdminBean;
import suishen.service.horoscope.meta.HoroscopeBean;

import javax.annotation.Resource;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;
import java.util.Objects;

/**
 * @auther: zhao tailen
 * @date: 2019/3/8 12:13
 */
@Slf4j
@Component
public class HoroscopeCache {

    private Cache<String, HoroscopeBean> horoscopeLocalCache = GuavaCache.callableCachedWithExpireAfterWrite(500, 3*Constant.ONE_MINUTE_SECONDS);

    @Resource
    private SuishenRedisTemplate coreRedisTemplate;

    @Resource
    private HoroscopeDao horoscopeDao;

    /**
     * 查询星座运势本地缓存
     *
     * @param type
     * @param day
     * @param timeType
     * @return
     */
    public HoroscopeBean getHoroscope(String type, String day, String timeType) {
        if (StringUtils.isBlank(type) || StringUtils.isBlank(day) || StringUtils.isBlank(timeType)) {
            return null;
        }

        try {
            HoroscopeBean bean = horoscopeLocalCache.get(type + day + timeType, () -> {
                HoroscopeBean horoscopeBean = getHoroscopeFromRedis(type, day, timeType);
                if (horoscopeBean == null) {
                    horoscopeBean = horoscopeDao.getHoroscope(type, day, timeType);
                    if (horoscopeBean != null) {
                        setHoroscope2Redis(horoscopeBean);
                    }

                }
                if (horoscopeBean != null) {
                    String regex = "<[^>]*>";
                    horoscopeBean.setLuckLove(horoscopeBean.getLuckLove().replaceAll(regex, "\n"));
                }
                return horoscopeBean != null ? horoscopeBean : new HoroscopeBean();
            });

            return bean != null && StringUtils.isNotBlank(bean.getDate()) ? bean : null;
        } catch (Exception e) {
            log.error("get horoscope from local cache error, type={}, day={}, timeType={}", type, day, timeType, e);
        }
        return null;
    }

    public boolean invalidate(String type, String day, String timeType) {
        if (StringUtils.isBlank(type) || StringUtils.isBlank(day) || StringUtils.isBlank(timeType)) {
            return false;
        }
        horoscopeLocalCache.invalidate(type + day + timeType);
        return true;
    }

    /**
     * 设置指定日期星座运势
     *
     * @param horoscope
     * @return
     */
    public boolean setHoroscope2Redis(HoroscopeBean horoscope) {
        return setHoroscopes2Redis(Arrays.asList(horoscope));
    }

    public boolean setHoroscopes2Redis(List<HoroscopeBean> horoscopeList) {
        if (CollectionUtils.isEmpty(horoscopeList)) {
            return false;
        }

        horoscopeList.stream()
                .filter(Objects::nonNull)
                .forEach(horoscope -> {
                    String cacheKey = RedisKey.keyOfDayHoroscope(horoscope.getType(), horoscope.getDate(), horoscope.getTimeType());
                    coreRedisTemplate.setex(cacheKey, calExpireSeconds(horoscope.getTimeType()), JSONUtil.getJsonString(horoscope));
                });
        return true;
    }

    /**
     * 将 更新 之后的星座数据刷入Redis
     *
     * @param bean 更新之后的星座数据
     * @return 存入结果
     * @date 2023/10/18
     */
    public boolean setHoroscopeAdmin2Redis(HoroscopeAdminBean bean){
        return setHoroscopesAdmin2Redis(Arrays.asList(bean));
    }
    public boolean setHoroscopesAdmin2Redis(List<HoroscopeAdminBean> horoscopeList) {
        if (CollectionUtils.isEmpty(horoscopeList)) {
            return false;
        }
        horoscopeList.stream()
                .filter(Objects::nonNull)
                .forEach(horoscope -> {
                    String cacheKey = RedisKey.keyOfDayHoroscope(horoscope.getType(), horoscope.getDay(), horoscope.getTimetype());
                    coreRedisTemplate.setex(cacheKey, calExpireSeconds(horoscope.getTimetype()), JSONUtil.getJsonString(horoscope));
                });
        return true;
    }

    private int calExpireSeconds(String timeType) {
        // 日：32天  周、月、年：2天（每天都会有一份）
        return StringUtils.equals(timeType, HoroscopeTimeTypeEnum.DAY.getType()) ? Constant.THIRTY_TWO_DAY_SECONDS : Constant.TWO_DAY_SECONDS;
    }

    /**
     * 获取指定日期星座运势
     *
     * @param type     星座类型
     * @param timeType 星座运势类型，day天 week周 month月 year年
     * @param date     日期，格式yyyyMMdd
     */
    public HoroscopeBean getHoroscopeFromRedis(String type, String date, String timeType) {
        String cacheKey = RedisKey.keyOfDayHoroscope(type, date, timeType);
        String horoscopeJson = coreRedisTemplate.get(cacheKey);
        return StringUtils.isBlank(horoscopeJson) ? null : JSONUtil.getObject(horoscopeJson, HoroscopeBean.class);
    }

    /**
     * 获取指定日期区间内的星座运势
     *
     * @param startDate
     * @param endDate
     * @param type
     * @param timeType
     * @return
     */
    public List<HoroscopeDayInfoDTO> getHoroscopesByDate(int startDate, int endDate, String type, String timeType) {
        String cacheKey = RedisKey.keyOfDaysHoroscopes(startDate, endDate, type, timeType);
        String horoscopesJson = coreRedisTemplate.get(cacheKey);
        return StringUtils.isBlank(horoscopesJson) ? Lists.newArrayList() : JSONUtil.getList(horoscopesJson, HoroscopeDayInfoDTO.class);
    }

    public boolean setHoroscopesByDate(int startDate, int endDate, String type, String timeType, List<HoroscopeDayInfoDTO> horoscopes) {
        String cacheKey = RedisKey.keyOfDaysHoroscopes(startDate, endDate, type, timeType);
        coreRedisTemplate.setex(cacheKey, Constant.ONE_HOUR_SECONDS, JSONUtil.getJsonString(horoscopes));
        return true;
    }
}

```
HoroscopeService
```java
package suishen.service.horoscope.service;

import com.google.common.collect.Lists;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.collections4.CollectionUtils;
import org.apache.commons.collections4.keyvalue.DefaultKeyValue;
import org.apache.commons.lang3.RandomUtils;
import org.apache.commons.lang3.StringUtils;
import org.apache.commons.lang3.math.NumberUtils;
import org.json.JSONObject;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import suishen.common.enums.DateFormatEnum;
import suishen.libs.util.JSONUtil;
import suishen.libs.web.exception.BusinessException;
import suishen.libs.web.meta.ActionStatus;
import suishen.service.city.meta.CityListBean;
import suishen.service.horoscope.cache.HoroscopeCache;
import suishen.service.horoscope.dao.HoroscopeAdminDao;
import suishen.service.horoscope.dao.HoroscopeDao;
import suishen.service.horoscope.dto.HoroscopeCalModuleDTO;
import suishen.service.horoscope.dto.HoroscopeDayInfoDTO;
import suishen.service.horoscope.dto.HoroscopeInfoDTO;
import suishen.service.horoscope.dto.HoroscopeSourceDTO;
import suishen.service.horoscope.enums.*;
import suishen.service.horoscope.meta.HoroscopeAdminBean;
import suishen.service.horoscope.meta.HoroscopeBean;
import suishen.service.horoscope.notify.HoroscopeDDNotifyService;
import suishen.service.horoscope.parser.HoroscopeV2Parser;
import suishen.support.utils.CopyUtil;
import suishen.support.utils.DateUtil;
import suishen.support.utils.ExceptionUtil;
import suishen.support.utils.LogUtil;
import suishen.weather.common.PageWrapper;
import suishen.weather.meta.city.AdminCityQueryResp;
import suishen.weather.meta.horoscope.AdminHoroscopeQueryParam;
import suishen.weather.meta.horoscope.AdminHoroscopeQueryResp;
import suishen.weather.meta.horoscope.AdminHoroscopeUpdateParam;

import javax.annotation.Resource;
import java.sql.SQLException;
import java.text.NumberFormat;
import java.text.ParseException;
import java.time.LocalDateTime;
import java.util.*;
import java.util.stream.Collectors;
import java.util.stream.Stream;

/**
 * @Auther: zhao tailen
 * @Date: 2019/3/1 11:06
 * @Description:
 */
@Slf4j
@Service
public class HoroscopeService {

    private static final String RESULT_JSON_KEY_DATE = "date";
    private static final String TODAY = "today";
    private static final String SRESULT_JSON_KEY_SIGNURL = "signURL";
    private static final String SIGN_NAME = "signName";

    private static final String BEST_FORTUNE_TIP = "今日%s最佳！";

    @Resource
    private HoroscopeCache horoscopeCache;

    @Resource
    private HoroscopeDao horoscopeDao;


    @Resource
    private HoroscopeV2Parser horoscopeV2Parser;

    @Autowired
    private HoroscopeDDNotifyService notifyService;

    public static final HoroscopeSourceDTO SOURCE_OF_ZIWEI = new HoroscopeSourceDTO("紫微黄历网", "https://m.ibazi.cn/?posid=zhwnl");


    public String escapeType(String type) {
        String signType;
        if (StringUtils.isBlank(type)) {
            signType = HoroscopeEnum.match(LocalDateTime.now()).getType();
        } else {
            signType = HoroscopeEnum.match(type)
                    .orElseGet(() -> HoroscopeEnum.match(LocalDateTime.now()))
                    .getType();
        }
        return signType;
    }


    /**
     * 获取星座运势信息
     *
     * @param type     星座
     * @param day      格式：yyyyMMdd
     * @param timeType 时间类型，如day、week、month、year
     * @return
     */
    public String getHoroscopeInfo(String type, String day, String timeType) {
        final String emptyJson = "{\"error\":\"暂无数据\"}";
        Optional<HoroscopeBean> horoscopeOptional = getHoroscope(type, day, timeType);
        return horoscopeOptional.map(HoroscopeBean::getJson).orElse(emptyJson);
    }

    public Optional<HoroscopeBean> getHoroscope(String type, String day, String timeType) {
        HoroscopeBean horoscope = null;
        try {
            horoscope = horoscopeCache.getHoroscope(type, day, timeType);
            if (horoscope == null) {
                log.error("get horoscope info fail, type={}, day={}, timeType={}", type, day, timeType);
                // 钉钉通知
                notifyService.sendHoroscopeNotExistNotify(type, day, timeType, null);
            }
        } catch (Exception e) {
            log.error("query horoscope from db error, type={}, day={}, timeType={}", type, day, timeType, e);
            notifyService.sendHoroscopeNotExistNotify(type, day, timeType, ExceptionUtil.getExceptionStackTraceInfo(e));
        }
        return Optional.ofNullable(horoscope);
    }

    /**
     * 获取星座运势信息（第三方）
     *
     * @param type     星座
     * @param day      日期，格式yyyyMMdd
     * @param timeType 时间类型
     * @return
     */
    public JSONObject getHoroscopeInfoForThird(String type, String day, String timeType) {
        // 参数校验
        this.checkHoroscopeInfoForThirdParams(type, day, timeType);

        HoroscopeBean bean = horoscopeCache.getHoroscope(type, day, timeType);
        if (bean == null) {
            // fetch from 紫微星座
            bean = horoscopeV2Parser.doParse(type, day, timeType);
            if (bean == null) {
                return null;
            }
            this.saveAndCache(Lists.newArrayList(bean));
        }
        return bean.toJSON();
    }

    private void checkHoroscopeInfoForThirdParams(String type, String day, String timeType) {
        if (StringUtils.isBlank(type) || StringUtils.isBlank(day) || StringUtils.isBlank(timeType)) {
            throw new BusinessException("参数错误");
        }
        if (HoroscopeUrlEnum.getByType(type) == null) {
            throw new BusinessException("星座类型错误");
        }
        if (HoroscopeTimeTypeEnum.getByType(timeType) == null) {
            throw new BusinessException("时间类型错误");
        }
        if (StringUtils.length(day) != 8) {
            throw new BusinessException("日期格式错误");
        }
    }

    /**
     * @param type        星座名称
     * @param fortuneList 运势类型
     * @param startDate   开始时间
     * @param endDate     结束时间
     * @complexity n
     * @Description 获取指定时间范围内的星座信息
     * @date: 2019/3/1信息
     */
    public List<HashMap<String, Object>> getHoroscopeInfoDTOListByDateV2(String startDate, String endDate, String type, List<String> fortuneList) throws Exception {

        List<HashMap<String, Object>> resultList = new ArrayList<>();
        List<String> betweenDate = DateUtil.getBetweenDate(startDate, endDate, DateFormatEnum.yyyyMMdd_LINE);
        for (int i = 0; i < betweenDate.size(); i++) {
            HashMap<String, Object> horoscopeInfoDTOHashMap = new HashMap<>();
            horoscopeInfoDTOHashMap.put(RESULT_JSON_KEY_DATE, betweenDate.get(i));
            HoroscopeUrlEnum horoscopeUrlEnum = HoroscopeUrlEnum.getByType(type);
            horoscopeInfoDTOHashMap.put(SIGN_NAME, horoscopeUrlEnum.getName());
            horoscopeInfoDTOHashMap.put(SRESULT_JSON_KEY_SIGNURL, HoroscopeUrlEnum.getUrlByType(type));
            String sevenDay = DateUtil.plusDay(7, DateFormatEnum.yyyyMMdd);

            for (int j = 0; j < fortuneList.size(); j++) {
                String date = DateUtil.getDate(betweenDate.get(i), DateFormatEnum.yyyyMMdd_LINE, DateFormatEnum.yyyyMMdd);
                String nextDay = DateUtil.plusDay(betweenDate.get(i), DateFormatEnum.yyyyMMdd_LINE, 1, DateFormatEnum.yyyyMMdd);
                boolean overAWeek = NumberUtils.toInt(date) > NumberUtils.toInt(sevenDay);
                switch (fortuneList.get(j)) {
                    case "tomorrow":
                        HoroscopeBean tomorrowDateResult = horoscopeCache.getHoroscope(type, nextDay, "day");
                        if (Objects.nonNull(tomorrowDateResult)) {
                            horoscopeInfoDTOHashMap.put(FortuneEnum.tomorrow.toString(), getHoroscopeInfoDTO(tomorrowDateResult));
                        }
                        break;
                    case "today":
                        HoroscopeBean todayDateResult = horoscopeCache.getHoroscope(type, date, "day");
                        if (Objects.nonNull(todayDateResult)) {
                            horoscopeInfoDTOHashMap.put(TODAY, getHoroscopeInfoDTO(todayDateResult));
                        }
                        break;
                    case "week":
                        HoroscopeBean weekDateResult = overAWeek ? null : horoscopeCache.getHoroscope(type, date, "week");
                        if (Objects.nonNull(weekDateResult)) {
                            horoscopeInfoDTOHashMap.put(weekDateResult.getTimeType(), getHoroscopeInfoDTO(weekDateResult));
                        }
                        break;
                    case "month":
                        HoroscopeBean monthDateResult = overAWeek ? null : horoscopeCache.getHoroscope(type, date, "month");
                        if (Objects.nonNull(monthDateResult)) {
                            horoscopeInfoDTOHashMap.put(monthDateResult.getTimeType(), getHoroscopeInfoDTO(monthDateResult));
                        }
                        break;
                    case "year":
                        HoroscopeBean yearDateResult = overAWeek ? null : horoscopeCache.getHoroscope(type, date, "year");
                        if (Objects.nonNull(yearDateResult)) {
                            horoscopeInfoDTOHashMap.put(yearDateResult.getTimeType(), getHoroscopeInfoDTO(yearDateResult));
                        }
                        break;
                    default:
                        break;
                }
            }
            resultList.add(horoscopeInfoDTOHashMap);
        }
        return resultList;

    }

    private HoroscopeInfoDTO getHoroscopeInfoDTO(HoroscopeBean horoscopeBean) {

        HoroscopeInfoDTO horoscopeInfoDTO = new HoroscopeInfoDTO();

        String date = "";
        if (StringUtils.isNotBlank(horoscopeBean.getDate())) {
            date = DateUtil.getDate(horoscopeBean.getDate(), DateFormatEnum.yyyyMMdd, DateFormatEnum.yyyyMMdd_LINE);
        }
        horoscopeInfoDTO.setDatetime(date);
        String indexHealth = horoscopeBean.getIndexHealth();
        if (StringUtils.isNotEmpty(checkStringNull(indexHealth))) {
            NumberFormat nf = NumberFormat.getPercentInstance();
            try {

                Number m = nf.parse(indexHealth);
                horoscopeInfoDTO.setHealthStar(m.floatValue() * 5);
            } catch (ParseException e) {
                log.error(e.getMessage(), e);
            }
        }
        horoscopeInfoDTO.setLoveDes(horoscopeBean.getLuckLove());
        horoscopeInfoDTO.setLoveStar(NumberUtils.toFloat(horoscopeBean.getIndexLove()));
        horoscopeInfoDTO.setLuckyColor(checkStringNull(horoscopeBean.getIndexColor()));
        horoscopeInfoDTO.setLuckyNumber(NumberUtils.toInt(horoscopeBean.getIndexNum()));
        horoscopeInfoDTO.setMatchSign(checkStringNull(horoscopeBean.getBestPartner()));
        horoscopeInfoDTO.setMoneyDes(horoscopeBean.getLuckMoney());
        horoscopeInfoDTO.setMoneyStar(NumberUtils.toFloat(horoscopeBean.getIndexInvest()));
        horoscopeInfoDTO.setSummaryDes(horoscopeBean.getDescription());
        horoscopeInfoDTO.setSummaryStar(NumberUtils.toFloat(horoscopeBean.getIndexTotal()));
        horoscopeInfoDTO.setWorkDes(horoscopeBean.getLuckWork());
        horoscopeInfoDTO.setWorkStar(NumberUtils.toFloat(horoscopeBean.getIndexWork()));
        horoscopeInfoDTO.setLuckyOrnt(checkStringNull(horoscopeBean.getDirection()));
        horoscopeInfoDTO.setStrength(checkStringNull(horoscopeBean.getAdvantage()));
        horoscopeInfoDTO.setWeakness(checkStringNull(horoscopeBean.getFaults()));
        horoscopeInfoDTO.setLuckyMonths(checkStringNull(horoscopeBean.getIndexMonth()));
        return horoscopeInfoDTO;
    }

    private String checkStringNull(String str) {
        if (StringUtils.isEmpty(str) || "-".equals(str) || "--".equals(str)) {
            return null;
        }
        return str;
    }


    /**
     * 获取指定日期的星座运势（只会从缓存取）
     *
     * <p>按照需求只会取当前月、下月的星座日运势，所以redis会缓存某日的星座运势一个月
     *
     * @param startDate 开始日期，格式yyyyMMdd
     * @param endDate   结束日期，格式yyyyMMdd
     * @param type      星座
     * @return
     */
    public List<HoroscopeDayInfoDTO> getDaysHoroscopeInfos(int startDate, int endDate, String type) {
        if (startDate > endDate) {
            throw new BusinessException("参数错误");
        }
        HoroscopeUrlEnum horoscopeUrlEnum = HoroscopeUrlEnum.getByType(type);
        if (horoscopeUrlEnum == null) {
            throw new BusinessException("参数错误");
        }

        // 缓存直接取
        String timeType = HoroscopeTimeTypeEnum.DAY.getType();
        List<HoroscopeDayInfoDTO> horoscopeList = horoscopeCache.getHoroscopesByDate(startDate, endDate, type, timeType);
        if (CollectionUtils.isNotEmpty(horoscopeList)) {
            return horoscopeList;
        }


        horoscopeList = Lists.newArrayList();
        for (int date = startDate; date <= endDate; ) {
            HoroscopeBean horoscope = horoscopeCache.getHoroscope(type, String.valueOf(date), timeType);
            if (horoscope == null) {
                horoscope = new HoroscopeBean();
                horoscope.setDate(String.valueOf(date));
                horoscope.setIndexLove("0");
                horoscope.setIndexWork("0");
                horoscope.setIndexInvest("0");

                notifyService.sendHoroscopeNotExistNotify(type, String.valueOf(date), timeType, null);
            }
            // build dto
            horoscopeList.add(this.buildHoroscopeDayInfoDTO(horoscope, horoscopeUrlEnum));

            date = DateUtil.plusDay(date, DateFormatEnum.yyyyMMdd, 1);
        }

        // 缓存
        horoscopeCache.setHoroscopesByDate(startDate, endDate, type, timeType, horoscopeList);

        return horoscopeList;
    }

    private HoroscopeDayInfoDTO buildHoroscopeDayInfoDTO(HoroscopeBean horoscope, HoroscopeUrlEnum starSignEnum) {
        HoroscopeUrlEnum matchStarSignEnum = HoroscopeUrlEnum.getByName(horoscope.getBestPartner());
        FortuneNameEnum bestFortuneEnum = calBestFortune(horoscope.getIndexLove(), horoscope.getIndexWork(), horoscope.getIndexInvest());
        return HoroscopeDayInfoDTO.builder()
                .date(horoscope.getDate())
                .best(bestFortuneEnum == null ? "" : bestFortuneEnum.getName())
                .bestDesc(bestFortuneEnum == null ? "" : String.format(BEST_FORTUNE_TIP, bestFortuneEnum.getDesc()))
                .starSign(horoscope.getType())
                .starSignName(starSignEnum.getName())
                .indexTotal(horoscope.getIndexTotal())
                .indexDesc(horoscope.getDescription())
                .luckColor(horoscope.getIndexColor())
                .luckNum(horoscope.getIndexNum())
                .matchStarSign(matchStarSignEnum == null ? "" : matchStarSignEnum.getType())
                .matchStarSignName(matchStarSignEnum == null ? "" : matchStarSignEnum.getName())
                .build();
    }

    private FortuneNameEnum calBestFortune(String love, String career, String fortune) {
        int loveStar = NumberUtils.toInt(love);
        int careerStar = NumberUtils.toInt(career);
        int fortuneStar = NumberUtils.toInt(fortune);

        int maxStar = NumberUtils.max(loveStar, careerStar, fortuneStar);
        if (maxStar < 4) {
            return null;
        }

        List<DefaultKeyValue<Integer, FortuneNameEnum>> bestFortuneList = Stream.of(
                        new DefaultKeyValue<>(loveStar, FortuneNameEnum.LOVE),
                        new DefaultKeyValue<>(careerStar, FortuneNameEnum.CAREER),
                        new DefaultKeyValue<>(fortuneStar, FortuneNameEnum.FORTUNE))
                .filter(e -> e.getKey() == maxStar)
                .collect(Collectors.toList());
        int randomIndex = RandomUtils.nextInt(0, bestFortuneList.size());
        return bestFortuneList.get(randomIndex).getValue();
    }

    public HoroscopeBean getHoroscopeFromDB(String type, String date, String timeType) {
        try {
            return horoscopeDao.getHoroscope(type, date, timeType);
        } catch (Exception e) {
            log.error("get Horoscope from db error, type={}, date={}, timeType={}", type, date, timeType, e);
        }
        return null;
    }

    /**
     * 更新 指定日期、指定类型 的星座运势
     *
     * @param timeType    时间类型
     * @param startDate   开始日期，格式：yyyyMMdd
     * @param endDate     结束日期，格式：yyyyMMdd
     * @param existIgnore true:若数据库中存在该星座运势，则不抓取更新 false：覆盖更新
     */
    public void updateHoroscopeByDate(String timeType, int startDate, int endDate, boolean existIgnore) {
        if (startDate > endDate) {
            return;
        }


        for (String type : HoroscopeUrlEnum.typeList) {
            updateHoroscopeByDate(type, timeType, startDate, endDate, existIgnore);
        }
    }

    /**
     * 更新指定日期、指定类型、指定星座的星座运势
     *
     * @param type        星座
     * @param timeType    时间类型
     * @param startDate   开始日期，格式：yyyyMMdd
     * @param endDate     结束日期，格式：yyyyMMdd
     * @param existIgnore true:若数据库中存在该星座运势，则不抓取更新 false：覆盖更新
     * @throws SQLException 数据库异常
     */
    private void updateHoroscopeByDate(String type, String timeType, int startDate, int endDate, boolean existIgnore) {
        if (startDate > endDate) {
            return;
        }

        List<HoroscopeBean> horoscopeList = Lists.newArrayList();

        if (HoroscopeTimeTypeEnum.isYear(timeType)) {
            // 数据库存在则不更新
            List<Integer> fetchDateList = Lists.newArrayList();
            if (existIgnore) {
                for (int date = startDate; date <= endDate; date = DateUtil.plusDay(date, DateFormatEnum.yyyyMMdd, 1)) {
                    HoroscopeBean horoscope = this.getHoroscopeFromDB(type, String.valueOf(startDate), timeType);
                    if (horoscope != null) {
                        continue;
                    }
                    fetchDateList.add(date);
                }
            } else {
                for (int date = startDate; date <= endDate; date = DateUtil.plusDay(date, DateFormatEnum.yyyyMMdd, 1)) {
                    fetchDateList.add(date);
                }
            }
            if (CollectionUtils.isEmpty(fetchDateList)) {
                return;
            }

            // fetch from 紫微星座
            HoroscopeBean horoscope = horoscopeV2Parser.doParse(type, String.valueOf(fetchDateList.get(0)), timeType);
            if (horoscope == null) {
                return;
            }

            horoscopeList = fetchDateList.stream()
                    .map(date -> {
                        HoroscopeBean temp = CopyUtil.transfer(horoscope, HoroscopeBean.class);
                        temp.setDate(String.valueOf(date));
                        return temp;
                    }).collect(Collectors.toList());
        } else {
            for (int date = startDate; date <= endDate; date = DateUtil.plusDay(date, DateFormatEnum.yyyyMMdd, 1)) {
                LogUtil.BUSINESS_LOG.info("========== start fetch horoscope, date={}, type={}, timeType={} ========== ", date, type, timeType);

                // 数据库存在则不更新
                HoroscopeBean horoscope;
                if (existIgnore) {
                    horoscope = this.getHoroscopeFromDB(type, String.valueOf(date), timeType);
                    if (horoscope != null) {
                        continue;
                    }
                }
                // fetch from 紫微星座
                horoscope = horoscopeV2Parser.doParse(type, String.valueOf(date), timeType);
                if (horoscope == null) {
                    continue;
                }
                horoscopeList.add(horoscope);
            }
        }

        if (CollectionUtils.isEmpty(horoscopeList)) {
            return;
        }

        // save db & cache
        this.saveAndCache(horoscopeList);
    }

    private void saveAndCache(List<HoroscopeBean> horoscopeList) {
        if (CollectionUtils.isEmpty(horoscopeList)) {
            return;
        }

        try {
            // db
            horoscopeDao.addBatch(horoscopeList);
            // 缓存
            horoscopeCache.setHoroscopes2Redis(horoscopeList);
        } catch (Exception e) {
            log.error("horoscope save to db or cache error, horoscopeList={}", JSONUtil.getJsonString(horoscopeList), e);
            // TODO 钉钉通知
        }
    }

    /**
     * 更新指定日期的星座运
     *
     * @param startDay
     * @param endDay
     * @return
     */
    public void reloadHoroscopeCache(int startDay, int endDay) {
        if (startDay == 0 || endDay == 0 || startDay > endDay) {
            return;
        }

        // 所有时间类型
        for (String timeType : HoroscopeTimeTypeEnum.timeTypeList) {
            boolean existIgnore = !HoroscopeTimeTypeEnum.isYear(timeType);
            updateHoroscopeByDate(timeType, startDay, endDay, existIgnore);
        }
    }

    /**
     * 查询星座运势（首页日历模块使用）
     *
     * @param type     星座，若为空，则查询所有星座
     * @param day      日期，格式：yyyyMMdd
     * @param timeType 时间类型，day week month year
     * @return
     */
    public HoroscopeCalModuleDTO getHoroscopeInfoForCalModule(String type, String day, String timeType) {
        HoroscopeUrlEnum horoscopeUrlEnum = HoroscopeUrlEnum.getByType(type.toLowerCase());
        if (horoscopeUrlEnum == null) {
            throw new BusinessException("不存在该星座类型哦");
        }
        if (StringUtils.length(day) != 8) {
            throw new BusinessException("日期类型参数错误");
        }
        if (StringUtils.isNotEmpty(timeType) && HoroscopeTimeTypeEnum.getByType(timeType) == null) {
            throw new BusinessException("时间类型参数错误");
        }

        timeType = StringUtils.isBlank(timeType) ? HoroscopeTimeTypeEnum.DAY.getType() : timeType;

        // 查询星座运势
        HoroscopeBean horoscope = horoscopeCache.getHoroscope(type, day, timeType);
        if (horoscope == null) {
            throw new BusinessException("星座运势不存在");
        }
        return this.buildHoroscopeCalModuleDTO(horoscope, horoscopeUrlEnum);
    }

    /**
     * 查询星座运势
     *
     * @param type     星座，若为空，则查询所有星座
     * @param day      日期，格式：yyyyMMdd
     * @param timeType 时间类型，day week month year
     * @return
     */
    public List<HoroscopeCalModuleDTO> getHoroscopeInfos(String type, String day, String timeType) {
        // 校验日期
        this.checkDate(day, timeType);

        List<String> typeList = StringUtils.isNotBlank(type) ? Lists.newArrayList(type.toLowerCase()) : HoroscopeUrlEnum.typeList;
        return typeList.stream()
                .map(e -> {
                    try {
                        HoroscopeUrlEnum horoscopeUrlEnum = HoroscopeUrlEnum.getByType(e);
                        if (horoscopeUrlEnum == null) {
                            return null;
                        }
                        HoroscopeBean horoscope = horoscopeCache.getHoroscope(type, day, timeType);
                        if (horoscope == null) {
                            return null;
                        }
                        return this.buildHoroscopeCalModuleDTO(horoscope, horoscopeUrlEnum);
                    } catch (Exception ex) {
                        log.error("query horoscope info error, type={}, day={}, timeType={}", e, day, timeType, ex);
                    }
                    return null;
                })
                .filter(Objects::nonNull)
                .collect(Collectors.toList());

    }

    private HoroscopeCalModuleDTO buildHoroscopeCalModuleDTO(HoroscopeBean horoscope, HoroscopeUrlEnum starSignEnum) {
        HoroscopeUrlEnum matchStarSignEnum = HoroscopeUrlEnum.getByName(horoscope.getBestPartner());
        return HoroscopeCalModuleDTO.builder()
                .date(horoscope.getDate())
                .starSign(horoscope.getType())
                .starSignName(starSignEnum.getName())
                .indexTotal(horoscope.getIndexTotal())
                .indexDesc(horoscope.getDescription())
                .luckColor(horoscope.getIndexColor())
                .luckNum(horoscope.getIndexNum())
                .matchStarSign(matchStarSignEnum == null ? "" : matchStarSignEnum.getType())
                .matchStarSignName(matchStarSignEnum == null ? "" : matchStarSignEnum.getName())
                .indexHealth(horoscope.getIndexHealth())
                .build();
    }

    /**
     * 校验时间
     * <p>每日运势，支持查询未来2个月
     * <p>周、月、年，支持查询未来10天
     *
     * @param day      日期，格式：yyyyMMdd
     * @param timeType 时间类型，day week month year
     */
    private void checkDate(String day, String timeType) {
        if (!NumberUtils.isCreatable(day)) {
            throw new BusinessException("日期参数错误");
        }

        int date = NumberUtils.toInt(day);
        // 校验未来
        int maxDate = 0;
        if (StringUtils.equalsIgnoreCase(timeType, HoroscopeTimeTypeEnum.DAY.getType())) {
            maxDate = DateUtil.plusMonths(2, DateFormatEnum.yyyyMMdd);
        } else {
            maxDate = DateUtil.plusDayInNum(10, DateFormatEnum.yyyyMMdd);
        }
        if (date > maxDate) {
            throw new BusinessException("日期超过查询范围");
        }
    }

    /**
     * 修复星座运势内容
     *
     * @param type
     * @param date
     * @param timeType
     * @return
     */
    public String fixHoroscopeContent(String type, String date, String timeType) throws SQLException {
        HoroscopeBean horoscope = horoscopeDao.getHoroscope(type, date, timeType);
        if (horoscope == null) {
            throw new BusinessException("数据不存在");
        }

        String indexColor = horoscope.getIndexColor();
        String newIndexColor = StringUtils.replace(indexColor, "蕃", "番");
        horoscope.setIndexColor(newIndexColor);

        this.saveAndCache(Lists.newArrayList(horoscope));
        horoscopeCache.invalidate(type, date, timeType);
        return horoscope.toString();
    }

    // ========================================== 定时任务 ==========================================


    /**
     * 抓取星座运势job
     *
     * @return
     */
    public void fetchHoroscopeJob() {
        long start = System.currentTimeMillis();
        LogUtil.BUSINESS_LOG.info("========== update all horoscope start ==========");

        try {
            // 所有时间类型，未来10天的数据
            int startDay = DateUtil.tomorrow();
            int endDay = DateUtil.plusDayInNum(10, DateFormatEnum.yyyyMMdd);
            for (String timeType : HoroscopeTimeTypeEnum.timeTypeList) {
                // 目前只有年数据会有覆盖更新
                boolean existIgnore = !HoroscopeTimeTypeEnum.isYear(timeType);
                updateHoroscopeByDate(timeType, startDay, endDay, existIgnore);
            }

            // 未来2月的日星座运势
            int futureStartDay = DateUtil.plusDayInNum(11, DateFormatEnum.yyyyMMdd);
            int futureEndDay = DateUtil.plusDayInNum(62, DateFormatEnum.yyyyMMdd);
            updateHoroscopeByDate(HoroscopeTimeTypeEnum.DAY.getType(), futureStartDay, futureEndDay, true);
        } catch (Exception e) {
            log.error("【HOROSCOPE JOB】fetch horoscope error", e);
        }
        LogUtil.BUSINESS_LOG.info("========== update all horoscope end, cost={}ms ==========", (System.currentTimeMillis() - start));
    }


}

```
消费者
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
http://dubbo.apache.org/schema/dubbo
http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="zhwnl_server"/>

    <!-- 使用zk注册中心暴露服务地址 -->
    <dubbo:registry address="zookeeper://node1.zk.all.platform.wtc.hwhosts.com:2181?backup=node2.zk.all.platform.wtc.hwhosts.com:2181,node3.zk.all.platform.wtc.hwhosts.com:2181"/>

    <dubbo:protocol name="dubbo" port="${dubbo.port}"/>


    <!-- Dubbo 消费者 -->
    <dubbo:consumer check="false"/>
    <dubbo:reference id="weatherHoroscopeRpcService" interface="suishen.weather.api.rpc.IWeatherHoroscopeService" check="false"
                   timeout="1000"/>
</beans>
```

RPC组件定义
```java
package suishen.remoterpc.weather.horoscope.service;

import com.google.common.collect.Lists;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import suishen.common.PageWrapper;
import suishen.esb.meta.RpcResult;
import suishen.remoterpc.common.BaseRpcComponent;
import suishen.remoterpc.weather.citylist.dto.AdminCityQueryRespDTO;
import suishen.weather.api.rpc.IWeatherHoroscopeService;
import suishen.weather.meta.city.AdminCityQueryRaram;
import suishen.weather.meta.horoscope.AdminHoroscopeQueryParam;
import suishen.weather.meta.horoscope.AdminHoroscopeQueryResp;
import suishen.weather.meta.horoscope.AdminHoroscopeUpdateParam;

import javax.annotation.Resource;
import java.util.ArrayList;
import java.util.List;

/**
 * 紫薇星座 - 数据维护rpc
 *
 * @author supanpan
 * @date 2023/10/18
 */
@Slf4j
@Service
public class HoroscopeComponent extends BaseRpcComponent {
    @Resource
    private IWeatherHoroscopeService weatherHoroscopeRpcService;

    /**
     * 查询紫薇星座数据列表
     *
     * @param param 查询参数
     * @return 结果
     */
    public PageWrapper<AdminHoroscopeQueryResp> queryHoroscopeList(AdminHoroscopeQueryParam param) {
        RpcResult<suishen.weather.common.PageWrapper<AdminHoroscopeQueryResp>> rpcResult =
                weatherHoroscopeRpcService.queryHoroscopeList(param);
        this.checkRpcResult(rpcResult);

        suishen.weather.common.PageWrapper<AdminHoroscopeQueryResp> rpcPageWrapper = rpcResult.getData();
        List<AdminHoroscopeQueryResp> rpcHoroscopeList = rpcPageWrapper.getList();

        return new PageWrapper<>(rpcPageWrapper.getPage(), rpcPageWrapper.getPageSize(),
                rpcPageWrapper.getTotalCount(), rpcHoroscopeList);
    }


    /**
     * 更新星座数据
     *
     * @param req 编辑参数
     * @return 更新结果
     */
    public boolean updateHoroscope(AdminHoroscopeUpdateParam req){
        RpcResult<Boolean> rpcResult = weatherHoroscopeRpcService.updateHoroscope(req);
        this.checkRpcResult(rpcResult);
        return rpcResult.getData();
    }

}

```
接口实现
```java
package suishen.access.admin.weather;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiParam;
import org.springframework.web.bind.annotation.*;
import suishen.common.PageWrapper;
import suishen.libs.meta.JSONResult;
import suishen.remoterpc.weather.citylist.dto.AdminCityQueryRespDTO;
import suishen.remoterpc.weather.citylist.dto.AdminCityUpdateReqDTO;
import suishen.remoterpc.weather.horoscope.service.HoroscopeComponent;
import suishen.weather.meta.city.AdminCityQueryRaram;
import suishen.weather.meta.horoscope.AdminHoroscopeQueryParam;
import suishen.weather.meta.horoscope.AdminHoroscopeQueryResp;
import suishen.weather.meta.horoscope.AdminHoroscopeUpdateParam;

import javax.annotation.Resource;

/**
 * 天气系统 - 紫薇星座后台接口
 *
 * @author supanpan
 * @date 2023/10/18
 */
@Api(value = "WEATHER 紫薇星座后台接口", tags = "HoroscopeAdminController", description = "紫薇星座后台接口")
@RestController
@RequestMapping("/backend/weather/horoscope")
public class WeatherHoroscopeAdminController {

    @Resource
    private HoroscopeComponent horoscopeComponent;

    @ApiOperation(value = "查询紫薇星座数据列表接口", response = AdminHoroscopeQueryResp.class, responseContainer = "Map")
    @GetMapping
    public JSONResult horoscopeList(@ApiParam(value = "时间类型 day week month year") @RequestParam(name = "time_type", required = false) String timeType,
                                    @ApiParam(value = "星座类型(英文)") @RequestParam(name = "type", required = false) String type,
                                    @ApiParam(value = "开始时间") @RequestParam(name = "start_date", required = false) String startDate,
                                    @ApiParam(value = "结束时间") @RequestParam(name = "end_date", required = false) String endDate,
                                    @ApiParam(value = "页次") @RequestParam(name = "page_num", defaultValue = "1", required = false) int pageNum,
                                    @ApiParam(value = "页大小") @RequestParam(name = "page_size", defaultValue = "10", required = false) int pageSize) {
        AdminHoroscopeQueryParam param = AdminHoroscopeQueryParam.builder()
                .type(type)
                .timetype(timeType)
                .startDate(startDate)
                .endDate(endDate)
                .pageNum(pageNum)
                .pageSize(pageSize)
                .build();
        PageWrapper<AdminHoroscopeQueryResp> result = horoscopeComponent.queryHoroscopeList(param);
        return JSONResult.okResult(result);
    }

    @PutMapping(value = "/{id}", headers = "Accept=application/json",produces = "application/json;charset=UTF-8", consumes = "application/json")
    @ApiOperation(value = "编辑紫薇星座数据接口")
    public JSONResult editHoroscope(@RequestBody AdminHoroscopeUpdateParam req) {
        return JSONResult.okResult(horoscopeComponent.updateHoroscope(req));
    }

}

```

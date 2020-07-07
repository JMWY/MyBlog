# 判断点和多边形关系

射线法判断点是否在多边形内
```c

bool SpatialUtil::isWithIn(float srcLat, float srcLng,
                            std::vector<float> lat_vec, std::vector<float> lng_vec, int pointNum)
{
    int leftTouch = 0;      //相交并且线在点的左侧次数
    float lat_cur, lat_next, lng_cur, lng_next;
    for (int i=0; i<pointNum; i++)
    {
        lat_cur = lat_vec[i];                                                                                                                          
        lng_cur = lng_vec[i];
        lat_next = (i+1)!=pointNum ? lat_vec[i+1] : lat_vec[0];
        lng_next = (i+1)!=pointNum ? lng_vec[i+1] : lng_vec[0];

        //如果纬度在两点之间进行计算
        if (((srcLat >lat_cur) && (srcLat < lat_next)) ||
            ((srcLat <lat_cur) && (srcLat > lat_next))) {
            if ((srcLng > lng_cur) && (srcLng > lng_next)){
                //相交并在右侧
                leftTouch++;
                continue;
            }   
            else if ((srcLng < lng_cur) && (srcLng < lng_next)){
                //相交并在左侧
                continue;
            }   
            else {
                float lngTouch = ( (lng_next - lng_cur) * (srcLat - lat_cur) / (lat_next - lat_cur) ) + lng_cur ; // 求相交点的经度
                if (srcLng > lngTouch){
                    leftTouch++;
                }   
            }   
        }  
        else if (srcLat == lat_next) {      //如果等于下一节点纬度
            if ( srcLng > lng_next){    //线在左侧才考虑
                    float lng_more = lng_vec[ (i % pointNum) ];
                    if (((lng_cur >= lng_next) && (lng_next > lng_more)) ||
                        ((lng_cur > lng_next) && (lng_next >= lng_more)) ||
                        ((lng_cur <= lng_next) && (lng_next < lng_more)) ||
                        ((lng_cur < lng_next) && (lng_next <= lng_more))){
                            leftTouch++;
                    }
            }
        }
    }
    if ((leftTouch % 2) == 0)     //有单次相交的是在范围内
        return false;
    return true;
}

```
点是否在矩形内
```c
bool SpatialUtil::isWithIn(float srcLat, float srcLng,
                            float dstLtLat, float dstLtLng, float dstRbLat, float dstRbLng)
{
    if ((srcLat <= dstLtLat) &&
        (srcLng >= dstLtLng) &&
        (srcLat >= dstRbLat) &&
        (srcLng <= dstRbLng))
        return true;
    return false;
}

```

地球经纬度计算距离
```c
#define PI 3.1415926535898
#define rad(d) (d*PI/180.0)
#define EARTH_RADIUS 6378.137                                                                                                                          

HA3_LOG_SETUP(amap, SpatialUtil);

float SpatialUtil::calcSpatial(float srcLat, float srcLng,
                                 float dstLat, float dstLng)
{
    double latDis = rad(srcLat) - rad(dstLat);
    double lonDis = rad(srcLng) - rad(dstLng);
    double s = 2 * asin(sqrt(pow(sin(latDis/2), 2) + cos(rad(srcLat)) * cos(rad(dstLat))
               * pow(sin(lonDis/2), 2))) * EARTH_RADIUS;
    s = round(s * 10000) / 10000;
    return s;
}

```


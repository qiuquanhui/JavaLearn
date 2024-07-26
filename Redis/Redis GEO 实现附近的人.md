# Redis GEO 实现附近的人

Redis GEO 是一种存储位置信息的数据结构，底层是 sort set 有序集合，可以根据经度，纬度计算出分数，并且算出各分数之间的距离。



我们可以使用 Redis GEO 来实现当前登录用户查找附近的人的功能。接下来我们一起实现



## 用户表添加字段

```sql
latitude DECIMAL(10, 8) NOT NULL,   //纬度
longitude DECIMAL(11, 8) NOT NULL,  //经度
```



## 实现接口

1. 使用 GEOAdd 上传所有用户的经度与纬度信息
2. 计算出当前登录用户与所有用户的距离
3. 计算出当前登录用户 1500 KM内的用户并按照距离降序排序。定义新接口。



1. 使用 GEOAdd 上传所有用户的经度与纬度信息

```java
  public void geoAddUser() {
        System.out.println("geoAddUser");
        //1.查询所有用户
        QueryWrapper<User> userQueryWrapper = new QueryWrapper<>();
        userQueryWrapper.isNotNull("longitude");
        userQueryWrapper.isNotNull("latitude");
        List<User> userList = userService.list(userQueryWrapper);

        //2.将 经度与纬度都不为空的用户插入到redis GEO 中
        if (userList == null){
            return;
        }
        List<RedisGeoCommands.GeoLocation<String>> locationList = new ArrayList<>(userList.size());
        for (User user : userList) {
            long id = user.getId();
            //long类型转化为String
            String idStr = String.valueOf(id);
            Point point = new Point(user.getLongitude(), user.getLatitude());
            RedisGeoCommands.GeoLocation<String> location = new RedisGeoCommands.GeoLocation<>(idStr, point);
            locationList.add(location);
        }
        stringRedisTemplate.opsForGeo().add(UserConstant.REDIS_GEO_KEY, locationList);
    }
```

插入之后打开 Redis 的客户端可以看到以下值

![img](https://cdn.nlark.com/yuque/0/2024/png/33747484/1718782460452-9177f314-2fdf-420e-9fa7-5d2791f9a06a.png)

2. 计算出当前登录用户与所有用户的距离

```java
  //    2. 计算出当前登录用户"15"为例，与所有用户的距离，将其加入到 返回用户信息的列表中
    @Test
    public void getDistance(){
        //2.计算当前登录用户与所有用户的距离
        // 2.1 查询所有用户
        QueryWrapper<User> userQueryWrapper = new QueryWrapper<>();
        userQueryWrapper.isNotNull("longitude");
        userQueryWrapper.isNotNull("latitude");
        List<User> userList = userService.list(userQueryWrapper);
        if (userList == null){
            return;
        }
        ArrayList<UserVO> userVOS = new ArrayList<>(userList.size());
        for (User user : userList){
            //登录用户以id = 15 为例
            double distance = stringRedisTemplate.opsForGeo().distance(UserConstant.REDIS_GEO_KEY, "15", String.valueOf(user.getId()), RedisGeoCommands.DistanceUnit.KILOMETERS).getValue();
            UserVO userVO = new UserVO();
            BeanUtils.copyProperties(user, userVO);
            userVO.setDistance(distance);
            userVOS.add(userVO);
        }
        //3.返回数据
        System.out.println(userVOS);
    }
```



3. 计算出当前登录用户 1500 KM内的用户并按照距离降序排序。定

```java
    //    3. 计算出当前登录用户 1500 KM内的用户并按照距离降序排序。定义新接口。
    @Test
    public void getRedius() {
        //    3. 计算出当前登录用户"15"为例 1500 KM内的用户并按照距离降序排序
        User loginUser = userService.getById(15);

        //创建距离
        Distance distance = new Distance(1500, RedisGeoCommands.DistanceUnit.KILOMETERS);

        //创建参数
        RedisGeoCommands.GeoRadiusCommandArgs geoRadiusCommandArgs = RedisGeoCommands.GeoRadiusCommandArgs.newGeoRadiusArgs().includeCoordinates().includeDistance().sort(Sort.Direction.ASC);

        //进行计算
        GeoResults<RedisGeoCommands.GeoLocation<String>> geoResults = stringRedisTemplate.opsForGeo().radius(UserConstant.REDIS_GEO_KEY,
                String.valueOf(loginUser.getId()), distance, geoRadiusCommandArgs);

        //遍历数据
        List<GeoResult<RedisGeoCommands.GeoLocation<String>>> content = geoResults.getContent();

        for (GeoResult<RedisGeoCommands.GeoLocation<String>> result : content) {
            RedisGeoCommands.GeoLocation<String> location = result.getContent();
            String id = location.getName();
            User user = userService.getById(Long.valueOf(id));
            UserVO userVO = new UserVO();
            BeanUtils.copyProperties(user, userVO);
            userVO.setDistance(result.getDistance().getValue());
            System.out.println(userVO);
        }
    }
```

## 定义新接口，返回登录用户查询附近的人。

```java
    @GetMapping("/searchNearUser")
    public BaseResponse<List<UserVO>> searchNearUser(Integer radius, HttpServletRequest request){
        if (radius == null ){
            throw new BusinessException(ErrorCode.PARAMS_ERROR);
        }

        return ResultUtils.success(userService.searchNearUser(radius, request));
    }

  @Override
    public List<UserVO> searchNearUser(Integer radius, HttpServletRequest request) {

        User loginUser = getLoginUser(request);

        if (loginUser == null){
            throw new BusinessException(ErrorCode.PARAMS_ERROR);
        }

        //创建距离
        Distance distance = new Distance(radius, RedisGeoCommands.DistanceUnit.KILOMETERS);

        //创建参数
        RedisGeoCommands.GeoRadiusCommandArgs geoRadiusCommandArgs = RedisGeoCommands.GeoRadiusCommandArgs.newGeoRadiusArgs().includeCoordinates().includeDistance().sort(Sort.Direction.ASC);

        //进行计算
        GeoResults<RedisGeoCommands.GeoLocation<String>> geoResults = stringRedisTemplate.opsForGeo().radius(UserConstant.REDIS_GEO_KEY,
                String.valueOf(loginUser.getId()), distance, geoRadiusCommandArgs);

        //遍历数据
        List<GeoResult<RedisGeoCommands.GeoLocation<String>>> content = geoResults.getContent();

        List<UserVO> userVos = new ArrayList<>(content.size());

        for (GeoResult<RedisGeoCommands.GeoLocation<String>> result : content) {
            RedisGeoCommands.GeoLocation<String> location = result.getContent();
            String id = location.getName();
            User user = getById(Long.valueOf(id));
            UserVO userVO = new UserVO();
            BeanUtils.copyProperties(user, userVO);
            userVO.setDistance(result.getDistance().getValue());
            userVos.add(userVO);
        }

        return userVos;
    }
```

这样使用 GEO 实现查询附近的人的功能就实现了。

首先要先上传所有用户的经度与纬度，然后使用 GEO 的命令，获取当前的登录用户的id，以及所有用户的id，用当前用户与计算用户进行距离的计算，得到距离 distance值。
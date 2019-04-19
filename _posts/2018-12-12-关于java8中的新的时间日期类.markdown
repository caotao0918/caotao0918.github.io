**关于java8中的新的时间日期类**

```java
    public static void main(String[] args) {
        Date date = new Date();
        LocalDate localDate = LocalDate.now();
        LocalTime localTime = LocalTime.now();
        LocalDateTime localDateTime = LocalDateTime.of(localDate,localTime);
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:SSS");
        DateTimeFormatter formatter2 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        String format = localDateTime.format(formatter);
        String format2 = localDateTime.format(formatter2);
        LocalDate plusDays = localDate.plusDays(2000);

        //将Date转换为Localdatetime对象,有两种方法
        Instant instant = date.toInstant();
        ZoneId zoneId = ZoneId.systemDefault();
        //方法1
        LocalDateTime localDateTime2 = instant.atZone(zoneId).toLocalDateTime();
        //方法2
        LocalDateTime localDateTime3 = LocalDateTime.ofInstant(instant, zoneId);

        //将localdatetime转换为date对象
        Date from = Date.from(LocalDateTime.of(LocalDate.now(), LocalTime.now()).plusSeconds(20000000).atZone(ZoneId.systemDefault()).toInstant());

        long time = date.getTime();
        System.out.println(plusDays);//2024-10-01
        System.out.println(localDate);//2019-04-11
        System.out.println(localDateTime);//2019-04-11T10:14:45.083
        System.out.println(date);//Thu Apr 11 10:05:38 CST 2019
        System.out.println(localTime);//10:07:33.806
        System.out.println(time);//1554949998807
        System.out.println(format);//2019-04-11 10:23:712   精确到毫秒   666
        System.out.println(format2);//2019-04-11 10:26:39
        System.out.println(localDateTime2);//2019-04-11T10:44:04.099
        System.out.println(localDateTime3);//2019-04-11T10:44:04.099
    }
```


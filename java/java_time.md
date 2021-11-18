LocalDate、LocalTime、LocalDateTime、Instant为不可变对象

```java
  public static void main(String[] args) {
    // 获取当前时间
    LocalDateTime now = LocalDateTime.now();

    // 获取时间戳
    long timestamp = now.atZone(ZoneOffset.systemDefault()).toInstant().toEpochMilli();
    System.out.println("timestamp: " + timestamp);

    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    System.out.println(now.format(formatter));

    // 解析
    LocalDateTime dateTime = LocalDateTime.parse("2021-11-18 17:49:16", formatter);
  }

```


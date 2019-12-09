### 从Request中获取调用方的ip和端口号
``` java
    /**
     * 获取ip
     * @param request
     * @return
     */
    private String getIpAddress(HttpServletRequest request) {
        String ip = request.getHeader("x-forwarded-for");
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("HTTP_CLIENT_IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("HTTP_X_FORWARDED_FOR");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getRemoteAddr();
        }
        if (ip == null || ip.length() == 0 || "0:0:0:0:0:0:0:1".equalsIgnoreCase(ip)) {
            ip = "127.0.0.1";
        }
        return ip;
    }
    
     /**
     * 获取端口号
     * @param request
     * @return
     */
    private Integer getIpPort(HttpServletRequest request) {
        return request.getServerPort();
    }
  
```
### 生成唯一id工具类(解决Java服务器时间回拨问题)
```java
  private volatile static SnowFlakeWorker snowFlakeWorkerInstance;

    // 1位标识部分    -      41位时间戳部分        -         10位节点部分     12位序列号部分
    /** 0 - 0000000000 0000000000 0000000000 0000000000 0 - 00000 - 00000 - 000000000000 */
    /**
     * 起始的时间戳
     */
    private final static long START_STMP = 1288834974657L;

    /**
     * 每一部分占用的位数
     */
    private final static long SEQUENCE_BIT = 12;  // 序列号占用的位数
    private final static long WORK_BIT = 10;    // 机器标识占用的位数

    /**
     * WORK_NUM最大值 1023
     */
    private final static long MAX_WORK_NUM = -1L ^ (-1L << WORK_BIT);
    /**
     * SEQUENCE最大值 4095
     */
    private final static long MAX_SEQUENCE = -1L ^ (-1L << SEQUENCE_BIT);

    /**
     * 每一部分向左的位移
     */
    private final static long WORK_LEFT = SEQUENCE_BIT;
    private final static long TIMESTMP_LEFT = WORK_LEFT + WORK_BIT;

    private long workId;
    private long sequence = 0L;  //序列号
    private long lastStmp = -1L; //上一次时间戳

    /** 步长, 1024 */
    private static long stepSize = 2 << 9;
    /** 基础序列号, 每发生一次时钟回拨即改变, basicSequence += stepSize */
    private long basicSequence = 0L;

    private SnowFlakeWorker(long workId) {
        if (workId > MAX_WORK_NUM || workId < 0) {
            throw new IllegalArgumentException("datacenterId can't be greater than MAX_DATACENTER_NUM or less than 0");
        }
        this.workId = workId;
    }


    protected synchronized static SnowFlakeWorker initSnowFlakeWorker(long workId) {
        snowFlakeWorkerInstance = new SnowFlakeWorker(workId);
        return snowFlakeWorkerInstance;
    }

    public static SnowFlakeWorker getInstance() {
        return snowFlakeWorkerInstance;
    }


    /**
     * 产生下一个ID
     */
    public synchronized long nextId() {
        long currStmp = getNewstmp();
        if (currStmp < lastStmp) {
            return handleClockBackwards(currStmp);
        }

        if (currStmp == lastStmp) {
            // 相同毫秒内，序列号自增
            sequence = (sequence + 1) & MAX_SEQUENCE;
            // 同一毫秒的序列数已经达到最大
            if (sequence == 0L) {
                currStmp = getNextMill();
            }
        } else {
            // 不同毫秒内，序列号置为 basicSequence
            sequence = basicSequence;
        }

        lastStmp = currStmp;

        return (currStmp - START_STMP) << TIMESTMP_LEFT  // 时间戳部分
                | workId << WORK_LEFT                    // 节点部分
                | sequence;                              // 序列号部分
    }

    /**
     * 处理时钟回拨
     */
    private long handleClockBackwards(long currStmp) {
        basicSequence += stepSize;
        if (basicSequence == MAX_SEQUENCE + 1) {
            basicSequence = 0;
            currStmp = getNextMill();
        }
        sequence = basicSequence;

        lastStmp = currStmp;

        return (currStmp - START_STMP) << TIMESTMP_LEFT  // 时间戳部分
                | workId << WORK_LEFT                    // 节点部分
                | sequence;                              // 序列号部分
    }

    private long getNextMill() {
        long mill = getNewstmp();
        while (mill <= lastStmp) {
            mill = getNewstmp();
        }
        return mill;
    }

    private long getNewstmp() {
        return System.currentTimeMillis();
    }


    public static void main(String[] args) {
        SnowFlakeWorker worker = new SnowFlakeWorker(1);
        for (int i = 0; i < 30; i++) {
            System.out.println(worker.nextId());
        }


    }
```


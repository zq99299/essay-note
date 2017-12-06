# 分片规则

除了官网提供的分片规则外，还提供了自定义规则，只不过在网上没有找到相关的配置方式,可能是因为太简单了。

想一想mycat是通过jar包方式启动的，配置文件里面写的分片规则类路径，那么就是说只要这个类在这个路径中被加载即可。

官网文档上也写了可以自定义规则，就是没有写方法。这里经过实验，下面是一个自定义分片规则的方法


步骤如下:

1. 创建一个普通的jar项目，里面引用 `Mycat-server-1.6.5-release.jar`这个包
2. 写一个java类，继承AbstractPartitionAlgorithm类
3. 打包该jar包，然后放到mycat/lib 目录下
4. 配置文件中就可以写上自定义规则的类了。

自定义规则可以参考官网已有的规则类。这里模仿了一个

通过联合字段分片：数字_日期； 定义成 1_2017 则可以同时满足业务规则数字1 又同时满足是2017年的数据分到一个片上。

```java
public class AutoPartitionByCombinationField extends AbstractPartitionAlgorithm {
    private String mapFile;
    private Map<String, Range> ranges = new HashMap<>();

    public void setMapFile(String mapFile) {
        this.mapFile = mapFile;
    }

    @Override
    public void init() {
        this.initialize();
    }

    private void initialize() {
        BufferedReader in = null;
        try {
            InputStream fin = this.getClass().getClassLoader().getResourceAsStream(this.mapFile);
            if (fin == null) {
                throw new RuntimeException("can't find class resource file " + this.mapFile);
            }
            in = new BufferedReader(new InputStreamReader(fin));

            String line = null;
            while ((line = in.readLine()) != null) {
                line = line.trim();
                if (line.startsWith("#") || line.startsWith("//")) {
                    continue;
                }
                int ind = line.indexOf("=");
                if (ind < 0) {
                    System.out.println(" warn: bad line int " + this.mapFile + " :" + line);
                } else {
                    String[] pairs = line.substring(0, ind).trim().split("_");
                    int valNum = Integer.parseInt(pairs[0].trim());
                    String valDate = pairs[1].trim();
                    int nodeId = Integer.parseInt(line.substring(ind + 1).trim());
                    Range range = new Range(valNum, valDate, nodeId);
                    ranges.put(range.key(), range);
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (in != null) {
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    @Override
    public Integer calculate(String columnValue) {
        // xx_xx 数字定点，
        System.out.println("columnValue = " + columnValue);
        if (columnValue == null || columnValue.trim().length() == 0) {
            throw new IllegalArgumentException("columnValue is null or empty");
        }
        String key = columnValue.trim();
        if (ranges.containsKey(key)) {
            return ranges.get(key).nodId;
        }
        throw new IllegalArgumentException("columnValue " + columnValue + " no match nodeId");
    }

    @Override
    public Integer[] calculateRange(String s, String s1) {
        return new Integer[0];
    }

    static class Range {
        public final int val;
        public final String valDate;
        public final int nodId;

        public Range(int val, String valDate, int nodId) {
            this.val = val;
            this.valDate = valDate;
            this.nodId = nodId;
        }

        public String key() {
            return val + "_" + valDate;
        }
    }
}
```
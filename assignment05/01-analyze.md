# Analyze and make aggregations.

# Aggregate Metrics By Sensor Processor
หลักการ:
การ Aggregation โดยเซ็นเซอร์ เป็นการรวบรวมข้อมูลจากแต่ละเซ็นเซอร์ที่เก็บข้อมูลเฉพาะตัว (เช่น อุณหภูมิ, ความชื้น, ความดัน, ฯลฯ) โดยการรวมข้อมูลจากหลายช่วงเวลา หลายตำแหน่ง หรือจากหลายเซ็นเซอร์ที่เกี่ยวข้องกัน
ใช้ในการสร้างสถิติพื้นฐาน เช่น ค่าเฉลี่ย (mean), ค่าต่ำสุด (min), ค่าสูงสุด (max), ความแปรปรวน (variance), หรือค่าเบี่ยงเบนมาตรฐาน (standard deviation)
ได้อะไร:
การวิเคราะห์ผลการทำงานของแต่ละเซ็นเซอร์ เช่น อุณหภูมิหรือความชื้นมีการแปรปรวนอย่างไรในช่วงเวลาใด ช่วยให้ตรวจสอบความผิดปกติของเซ็นเซอร์ได้
ทราบถึงความน่าเชื่อถือของข้อมูลจากเซ็นเซอร์แต่ละตัว และสามารถระบุได้ว่าเซ็นเซอร์ใดทำงานผิดปกติ

@Component
public class AggregateMetricsBySensorProcessor {

    private static final Logger logger = LoggerFactory.getLogger(AggregateMetricsBySensorProcessor.class);

    private final static int WINDOW_SIZE_IN_MINUTES = 5;
    private final static String WINDOW_STORE_NAME = "aggregate-metrics-by-sensor-tmp";

    /**
     * Agg Metrics Sensor Topic Output
     */
    @Value("${kafka.topic.aggregate-metrics-sensor}")
    private String aggMetricsSensorOutput;

    /**
     *
     * @param stream
     */
    public void process(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        buildAggregateMetricsBySensor(stream)
                .to(aggMetricsSensorOutput, Produced.with(String(), new SensorAggregateMetricsSensorSerde()));
    }

    /**
     * Build Aggregate Metrics By Sensor Stream
     *
     * @param stream
     * @return
     */
    private KStream<String, SensorAggregateSensorMetricsDTO> buildAggregateMetricsBySensor(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        return stream
                .map((key, val) -> new KeyValue<>(val.getId(), val))
                .groupByKey(Grouped.with(String(), new SensorDataSerde()))
                .windowedBy(TimeWindows.of(Duration.ofMinutes(WINDOW_SIZE_IN_MINUTES)).grace(Duration.ofMillis(0)))
                .aggregate(SensorAggregateSensorMetricsDTO::new,
                        (String k, SensorDataDTO v, SensorAggregateSensorMetricsDTO va) -> aggregateData(v, va),
                         buildWindowPersistentStore()
                )
                .suppress(Suppressed.untilWindowCloses(unbounded()))
                .toStream()
                .map((key, value) -> KeyValue.pair(key.key(), value));
    }

    /**
     * Build Window Persistent Store
     *
     * @return
     */
    private Materialized<String, SensorAggregateSensorMetricsDTO, WindowStore<Bytes, byte[]>> buildWindowPersistentStore() {
        return Materialized
                .<String, SensorAggregateSensorMetricsDTO, WindowStore<Bytes, byte[]>>as(WINDOW_STORE_NAME)
                .withKeySerde(String())
                .withValueSerde(new SensorAggregateMetricsSensorSerde());
    }

    /**
     * Aggregate Data
     *
     * @param v
     * @param va
     * @return
     */
    private SensorAggregateSensorMetricsDTO aggregateData(final SensorDataDTO v, final SensorAggregateSensorMetricsDTO va) {
        // Sensor Data
        va.setId(v.getId());
        // Sensor Data
        va.setId(v.getId());
        va.setName(v.getName());
        // Start Agg
        if (va.getStartAgg() == null) {
            final Date startAggAt = new Date();
            va.setStartAgg(startAggAt);
            va.setStartAggTm(startAggAt.getTime());
        }
        va.setCountMeasures(va.getCountMeasures() + 1);
        // Temperature
        va.setSumTemperature(va.getSumTemperature() + v.getPayload().getTemperature());
        va.setAvgTemperature(va.getSumTemperature() / va.getCountMeasures()); // Humidity
        // Humidity
        va.setSumHumidity(va.getSumHumidity() + v.getPayload().getHumidity());
        va.setAvgHumidity(va.getSumHumidity() / va.getCountMeasures()); // Luminosity
        // Luminosity
        va.setSumLuminosity(va.getSumLuminosity() + v.getPayload().getLuminosity());
        va.setAvgLuminosity(va.getSumLuminosity() / va.getCountMeasures()); // Pressure
        // Pressure
        va.setSumPressure(va.getSumPressure() + v.getPayload().getPressure());
        va.setAvgPressure(va.getSumPressure() / va.getCountMeasures());

        // End Agg
        final Date endAggAt = new Date();
        va.setEndAgg(endAggAt);
        va.setEndAggTm(endAggAt.getTime());
        return va;
    }

}

ถูกสร้างขึ้นโดยใช้ Apache Kafka Streams เพื่อประมวลผลข้อมูลสตรีมจากเซนเซอร์ โดยทำการจัดกลุ่มข้อมูลตาม sensor ID และคำนวณค่าเฉลี่ยและผลรวมของข้อมูล เช่น อุณหภูมิ ความชื้น ความสว่าง และความดัน ในช่วงเวลาที่กำหนด (5 นาที) โดยใช้ฟังก์ชัน windowedBy() สำหรับการแบ่งช่วงเวลา (windowing) และ aggregate() สำหรับการคำนวณค่าต่าง ๆ หลังจากที่ทำการคำนวณเสร็จแล้ว ผลลัพธ์จะถูกส่งไปยัง Kafka topic อื่นที่กำหนดไว้เพื่อการจัดเก็บหรือการประมวลผลเพิ่มเติม

# Aggregate Metrics By Place Processor
หลักการ:
การ Aggregation โดยสถานที่ หมายถึงการรวมข้อมูลจากเซ็นเซอร์ทั้งหมดที่อยู่ในสถานที่หรือพื้นที่เฉพาะ เช่น อาคาร, โรงงาน, หรือห้อง
ข้อมูลจากเซ็นเซอร์หลายตัวที่ติดตั้งอยู่ในพื้นที่เดียวกันจะถูกรวบรวมและวิเคราะห์ร่วมกัน ซึ่งอาจจะเป็นการวิเคราะห์ข้อมูลเชิงพื้นที่ (Spatial Data)
ได้อะไร:
สามารถวิเคราะห์ผลการทำงานหรือสภาพแวดล้อมในสถานที่ใดสถานที่หนึ่งได้ เช่น อุณหภูมิเฉลี่ยหรือความชื้นเฉลี่ยในอาคาร มีแนวโน้มการเปลี่ยนแปลงอย่างไร
ช่วยในการทำความเข้าใจภาพรวมของสภาพแวดล้อมในพื้นที่นั้น ๆ โดยอิงจากข้อมูลของเซ็นเซอร์ทั้งหมดที่อยู่ในพื้นที่

@Component
public class AggregateMetricsByPlaceProcessor {

    private static final Logger logger = LoggerFactory.getLogger(AggregateMetricsByPlaceProcessor.class);

    private final static int WINDOW_SIZE_IN_MINUTES = 5;
    private final static String WINDOW_STORE_NAME = "aggregate-metrics-by-place-tmp";

    /**
     * Agg Metrics Place Topic Output
     */
    @Value("${kafka.topic.aggregate-metrics-place}")
    private String aggMetricsPlaceOutput;

    /**
     *
     * @param stream
     */
    public void process(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        buildAggregateMetrics(stream)
                .to(aggMetricsPlaceOutput, Produced.with(String(), new SensorAggregateMetricsPlaceSerde()));
    }

    /**
     * Build Aggregate Metrics Stream
     *
     * @param stream
     * @return
     */
    private KStream<String, SensorAggregatePlaceMetricsDTO> buildAggregateMetrics(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        return stream
                .map((key, val) -> new KeyValue<>(val.getPlaceId(), val))
                .groupByKey(Grouped.with(String(), new SensorDataSerde()))
                .windowedBy(TimeWindows.of(Duration.ofMinutes(WINDOW_SIZE_IN_MINUTES)).grace(Duration.ofMillis(0)))
                .aggregate(SensorAggregatePlaceMetricsDTO::new,
                        (String k, SensorDataDTO v, SensorAggregatePlaceMetricsDTO va) -> aggregateData(v, va),
                        buildWindowPersistentStore()
                )
                .suppress(Suppressed.untilWindowCloses(unbounded()))
                .toStream()
                .map((key, value) -> KeyValue.pair(key.key(), value));
    }

    /**
     * Build Window Persistent Store
     *
     * @return
     */
    private Materialized<String, SensorAggregatePlaceMetricsDTO, WindowStore<Bytes, byte[]>> buildWindowPersistentStore() {
        return Materialized
                .<String, SensorAggregatePlaceMetricsDTO, WindowStore<Bytes, byte[]>>as(WINDOW_STORE_NAME)
                .withKeySerde(String())
                .withValueSerde(new SensorAggregateMetricsPlaceSerde());
    }

    /**
     * Aggregate Data
     *
     * @param v
     * @param va
     * @return
     */
    private SensorAggregatePlaceMetricsDTO aggregateData(final SensorDataDTO v, final SensorAggregatePlaceMetricsDTO va) {
        va.setPlaceId(v.getId());
        // Start Agg
        if (va.getStartAgg() == null) {
            final Date startAggAt = new Date();
            va.setStartAgg(startAggAt);
            va.setStartAggTm(startAggAt.getTime());
        }
        va.setCountMeasures(va.getCountMeasures() + 1);
        // Temperature
        va.setSumTemperature(va.getSumTemperature() + v.getPayload().getTemperature());
        va.setAvgTemperature(va.getSumTemperature() / va.getCountMeasures()); // Humidity
        // Humidity
        va.setSumHumidity(va.getSumHumidity() + v.getPayload().getHumidity());
        va.setAvgHumidity(va.getSumHumidity() / va.getCountMeasures()); // Luminosity
        // Luminosity
        va.setSumLuminosity(va.getSumLuminosity() + v.getPayload().getLuminosity());
        va.setAvgLuminosity(va.getSumLuminosity() / va.getCountMeasures()); // Pressure
        // Pressure
        va.setSumPressure(va.getSumPressure() + v.getPayload().getPressure());
        va.setAvgPressure(va.getSumPressure() / va.getCountMeasures());

        // End Agg
        final Date endAggAt = new Date();
        va.setEndAgg(endAggAt);
        va.setEndAggTm(endAggAt.getTime());
        return va;
    }

}
โปรเซสเซอร์ Aggregate Metrics By Place Processor ทำหน้าที่รวมข้อมูลในลักษณะเดียวกับ Aggregate Metrics By Sensor Processor แต่ในกรณีนี้ การคำนวณค่าเฉลี่ยจะอ้างอิงจาก สถานที่ (Place ID) แทนที่จะเป็นเซ็นเซอร์แต่ละตัว

# Aggregate Metrics Time Series
หลักการ:
การ Aggregation แบบ Time Series (อนุกรมเวลา) เป็นการรวมข้อมูลโดยอิงตามเวลา ซึ่งเป็นการสรุปข้อมูลในช่วงเวลาต่าง ๆ เช่น รายวัน รายสัปดาห์ หรือรายชั่วโมง
ข้อมูลจากเซ็นเซอร์จะถูกจัดกลุ่มตามเวลา แล้วคำนวณค่าสถิติ เช่น ค่าเฉลี่ยหรือแนวโน้ม (trend) ในช่วงเวลานั้น ๆ เพื่อดูการเปลี่ยนแปลงของข้อมูลตามกาลเวลา
ได้อะไร:
ช่วยในการระบุแนวโน้มของข้อมูลในระยะยาว เช่น อุณหภูมิสูงขึ้นในช่วงฤดูร้อนหรือลดลงในช่วงกลางคืน
สามารถดูพฤติกรรมที่เปลี่ยนแปลงไปตามเวลาหรือเหตุการณ์เฉพาะ และสามารถช่วยให้เกิดการวางแผนล่วงหน้า หรือทำให้สามารถคาดการณ์เหตุการณ์ในอนาคตได้

@Component
public class MetricsTimeSeriesProcessor {

    private static final Logger logger = LoggerFactory.getLogger(MetricsTimeSeriesProcessor.class);

    private final static String SENSOR_TIME_SERIE_NAME = "sample_sensor_metric";
    private final static String SENSOR_TIME_SERIE_TYPE = "sensor";

    /**
     * Metrics Time Series
     */
    @Value("${kafka.topic.metrics-time-series}")
    private String metricTimeSeriesOutput;

    /**
     *
     * @param stream
     */
    public void process(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        stream
                .map((key, val) -> new KeyValue<>(val.getId(), buildSensorTimeSerieMetric(val)))
                .to(metricTimeSeriesOutput, Produced.with(String(), new SensorTimeSerieMetricSerde()));
    }

    /**
     * Build Sensor Time Serie Meter
     *
     * @param sensorData
     * @return
     */
    private SensorTimeSerieMetricDTO buildSensorTimeSerieMetric(final SensorDataDTO sensorData) {
        return SensorTimeSerieMetricDTO.builder()
                .name(SENSOR_TIME_SERIE_NAME)
                .timestamp(new Date().getTime())
                .type(SENSOR_TIME_SERIE_TYPE)
                .dimensions(SensorTimeSerieMetricDimensionsDTO.builder()
                        .placeId(sensorData.getPlaceId())
                        .sensorId(sensorData.getId())
                        .sensorName(sensorData.getName())
                        .build())
                .values(SensorTimeSerieMetricValuesDTO.builder()
                        .humidity((double) sensorData.getPayload().getHumidity())
                        .luminosity((double) sensorData.getPayload().getLuminosity())
                        .pressure((double) sensorData.getPayload().getPressure())
                        .temperature((double) sensorData.getPayload().getTemperature())
                        .build())
                .build();
    }

}

โปรเซสเซอร์ Aggregate Metrics Time Series Processor มีจุดประสงค์ในการแปลงข้อมูลให้เหมาะสมกับการใช้งานใน Prometheus โดยใช้ชื่อเซ็นเซอร์, รหัสเซ็นเซอร์ และตัวระบุสถานที่เป็น มิติข้อมูล (dimensions) ในการจัดการข้อมูลเหล่านี้

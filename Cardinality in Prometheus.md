# Cardinality in Prometheus

### Key Points About Cardinality in Prometheus

**Cardinality** in Prometheus refers to the number of unique time series in your data set, which is determined by the unique combinations of labels for each metric. It can significantly impact performance, resource consumption, and query efficiency in Prometheus.

Here’s a breakdown of cardinality in Prometheus:

1. **Metrics and Labels**: 
   - Each metric in Prometheus can have labels (like `job`, `instance`, `region`, etc.). A combination of a metric name and its label values creates a unique time series.
   - Example: If you have a metric `http_requests_total` with labels `method` and `status_code`, and each label can have several possible values, the cardinality grows quickly.
   
2. **High Cardinality**:
   - When there are many unique combinations of labels (high cardinality), the number of time series increases.
   - High cardinality can lead to performance problems, such as:
     - **Increased memory usage**: Storing many time series consumes more memory and resources.
     - **Slower queries**: More time series means longer query execution times, especially with complex aggregations.

3. **Low Cardinality**:
   - Low cardinality means fewer unique combinations of labels, which is easier to manage and scale. Queries perform faster and consume fewer resources.

4. **Impact of Cardinality**:
   - **Resource Usage**: Each unique combination of label values creates a new time series, meaning high cardinality leads to more storage, memory, and CPU consumption.
   - **Query Efficiency**: High cardinality can lead to slow query performance, especially when using functions like `sum()`, `avg()`, or `rate()` that need to process a large number of time series.
   - **Scrape Overhead**: More time series means more data to scrape and store, which could slow down your Prometheus scraping process.

### Business Use Cases & Examples

1. **Web Application Metrics**:
   - **Use Case**: Monitoring HTTP request counts and response times for a web application.
   - **Example**: 
     - Metric: `http_requests_total`
     - Labels: `method` (GET, POST, PUT), `status_code` (200, 404, 500), `endpoint` (login, checkout, home)
     - Cardinality: For a large app with many endpoints and status codes, this could result in a high cardinality (e.g., 100s or 1000s of unique time series).

   **Issue**: A large number of combinations (high cardinality) could strain Prometheus, causing slower queries and higher memory usage.

   **Solution**: Limit the number of labels or adjust the granularity of metrics (e.g., aggregate by status code only, not by endpoint).

2. **Microservices Monitoring**:
   - **Use Case**: Monitoring the performance of individual microservices in a large system.
   - **Example**: 
     - Metric: `service_latency_seconds`
     - Labels: `service_name` (user-service, order-service), `method` (GET, POST), `response_code` (200, 500)
     - Cardinality: In a large microservices environment with many services and operations, this can easily lead to a high cardinality.

   **Issue**: Each combination of service and method generates a time series, leading to an explosion in the number of time series.

   **Solution**: You could sample specific metrics (e.g., latency only for critical paths) or use a reduced set of labels, such as grouping by `service_name` instead of by every HTTP method.

3. **E-Commerce Platform Metrics**:
   - **Use Case**: Tracking product views and orders on an e-commerce website.
   - **Example**: 
     - Metric: `product_views_total`
     - Labels: `product_id`, `user_region`, `platform` (mobile, desktop)
     - Cardinality: If your platform has thousands of products, combined with many regions and platform types, the cardinality could quickly become large.

   **Issue**: A huge number of products results in a large number of time series, making it difficult to scale Prometheus efficiently.

   **Solution**: Implement aggregation by product categories or user regions instead of tracking each individual product.

4. **Cloud Infrastructure Monitoring**:
   - **Use Case**: Monitoring resources like CPU and memory usage across different instances in a cloud environment.
   - **Example**:
     - Metric: `cpu_usage_seconds_total`
     - Labels: `instance_id`, `region`, `zone`, `machine_type`
     - Cardinality: With dynamic scaling in cloud environments, new instances can spin up rapidly, each adding to the number of time series.

   **Issue**: If you're monitoring hundreds or thousands of instances, especially with multiple labels (like `region` or `zone`), it can quickly increase the number of time series Prometheus has to manage.

   **Solution**: Consider using labels that represent logical groupings, like `environment` (production, staging), instead of tracking every instance individually.

5. **Network Monitoring**:
   - **Use Case**: Tracking the number of packets or bytes transmitted across a network.
   - **Example**:
     - Metric: `network_bytes_sent_total`
     - Labels: `interface`, `host`, `direction` (in, out)
     - Cardinality: If your network has many interfaces across many hosts, the number of time series could grow rapidly.

   **Issue**: High cardinality with multiple network interfaces could overwhelm Prometheus if not managed carefully.

   **Solution**: Aggregate metrics at the interface or host level, or reduce the frequency of scraping for metrics with high cardinality.

### Best Practices for Managing Cardinality

1. **Use fewer labels**: Avoid using labels that have a high cardinality, like `user_id`, `session_id`, or `product_id` unless absolutely necessary.
2. **Group related metrics**: For instance, aggregate by service type rather than individual instances or specific operations.
3. **Limit cardinality of dynamic labels**: Labels like `instance_id` or `hostname` can change frequently in cloud environments, leading to high cardinality. Consider limiting the scope of these labels or using higher-level aggregations.
4. **Prometheus Relabeling**: Use relabeling to adjust labels before storing data. This can help reduce unnecessary cardinality by changing how data is scraped and stored.
5. **Long-term data aggregation**: For historical data, consider aggregating old metrics over time to reduce cardinality without losing valuable insight.

### Conclusion

Cardinality is an important factor to consider when working with Prometheus. By understanding the effects of high cardinality on memory usage, query performance, and scalability, you can design your metrics and labels more effectively. Managing cardinality helps ensure that your Prometheus setup can handle the load of large-scale systems efficiently without performance degradation.

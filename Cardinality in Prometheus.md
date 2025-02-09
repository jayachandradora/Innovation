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

# Example with Calculation 

Let's break down the calculation of **cardinality** and how it impacts Prometheus in your example of **web application metrics**.

### Scenario Overview:
You are monitoring the metric `http_requests_total` with the following labels:

- `method`: (GET, POST, PUT)
- `status_code`: (200, 404, 500)
- `endpoint`: (login, checkout, home)

### Cardinality Calculation:

The **cardinality** is determined by multiplying the number of unique possible values for each label. In this case:

- `method` has 3 possible values: GET, POST, PUT.
- `status_code` has 3 possible values: 200, 404, 500.
- `endpoint` has 3 possible values: login, checkout, home.

So, the total number of unique combinations (unique time series) would be:

\[
\text{Cardinality} = \text{Number of methods} \times \text{Number of status codes} \times \text{Number of endpoints}
\]

\[
\text{Cardinality} = 3 \times 3 \times 3 = 27
\]

This results in **27 unique time series** (combinations of method, status code, and endpoint) that Prometheus has to scrape, store, and query.

### Impact of High Cardinality:

Now, imagine the number of endpoints or the number of status codes grows. For instance:

- If you add more status codes (e.g., 5 different status codes instead of 3), the cardinality increases.

### New Cardinality with More Status Codes:
Suppose `status_code` now has 5 values: 200, 400, 404, 500, 502.

- `method` still has 3 values.
- `status_code` now has 5 values.
- `endpoint` still has 3 values.

\[
\text{Cardinality} = 3 \times 5 \times 3 = 45
\]

This results in **45 unique time series**.

### Example with More Labels (Higher Cardinality):

If your application has more endpoints (say, 10 different endpoints) or more HTTP methods (e.g., PUT, DELETE, PATCH), the number of combinations grows dramatically:

- 3 methods
- 5 status codes
- 10 endpoints

\[
\text{Cardinality} = 3 \times 5 \times 10 = 150
\]

Now, you're dealing with **150 unique time series** that Prometheus needs to scrape, store, and query.

### Performance Implications of High Cardinality:

1. **Memory Usage**:
   - Prometheus stores each time series independently, so higher cardinality means more time series to store, which increases memory usage. Each time series consumes memory for storing data points over time.
   - Example: If each time series stores 100 data points per minute for a month, that's **100 points * 150 time series = 15,000 data points**. Multiply that across a year, and you can see how quickly the storage requirements grow.

2. **Scraping Overhead**:
   - Prometheus needs to scrape each of these 150 time series periodically. If you have a high-frequency scrape interval (e.g., every 15 seconds), that results in significant overhead in terms of both the Prometheus server’s CPU usage and network bandwidth.
   - Example: If Prometheus scrapes 150 different combinations of time series every 15 seconds, that results in **150 * 4 = 600 requests per minute** (since there are 4 scrape intervals in one minute). That’s a lot of HTTP requests, and it adds to Prometheus' scrape time and memory usage.

3. **Slower Queries**:
   - With high cardinality, querying metrics that involve aggregation, filtering, or transformations becomes slower. Prometheus will need to scan and aggregate across a larger number of time series, which can increase the query time.
   - Example: If you're querying the sum of `http_requests_total` for a particular `status_code` (e.g., 404), Prometheus must scan all the time series that have `status_code=404` and aggregate them, which is slower when there are many combinations.

### How It Strains Prometheus:
- **Prometheus stores each combination as a separate time series**, and each time series consumes memory and processing power.
- As cardinality increases, **Prometheus' query performance degrades** because the system has to handle more data.
- High cardinality can also lead to issues such as **disk I/O spikes** as Prometheus tries to read and write large amounts of time series data.

### How to Mitigate Cardinality Issues:
1. **Limit the Number of Labels**:
   - Use fewer labels or aggregate over higher-level categories (e.g., aggregate by `status_code` only, instead of tracking every individual `endpoint`).
   
2. **Reduce Granularity**:
   - Avoid using dynamic values (like user IDs or session IDs) as labels. Instead, group data into more stable categories (e.g., by service or endpoint type).

3. **Apply Relabeling**:
   - Use Prometheus relabeling rules to adjust how metrics are scraped. For instance, you could combine endpoints into fewer categories to reduce cardinality.

4. **Use a Longer Scrape Interval**:
   - For less critical metrics, increase the scrape interval to reduce the overhead on Prometheus.

By managing and controlling cardinality in your web application metrics, you ensure that Prometheus can efficiently handle your metrics without running into memory, query, or storage issues.

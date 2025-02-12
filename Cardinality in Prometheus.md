# Prometheus stores time series data in a time series database (TSDB)

Prometheus stores time series data in a time series database (TSDB) in a way that allows efficient querying and retrieval of metrics. The way Prometheus handles data, especially in terms of **cardinality** (i.e., unique combinations of label values), is crucial for understanding how it manages large amounts of time series data.

Here's how Prometheus stores time series data with cardinality combinations:

### 1. **Time Series Data Structure**
A time series in Prometheus is defined by the **metric name** and a set of **labels** (and their values). For example, if you have a metric like `http_requests_total`, it might be associated with labels such as `method`, `status_code`, and `region`.

Each combination of the **metric name** and **labels** represents a **unique time series**. So, a time series in Prometheus is identified by:
- **Metric Name**: The metric name, such as `http_requests_total`.
- **Labels**: The label set (e.g., `{method="GET", status_code="200", region="us-west"}`).

#### Example:
For `http_requests_total`, Prometheus would store multiple time series depending on the combination of label values:
- `http_requests_total{method="GET", status_code="200", region="us-west"}`
- `http_requests_total{method="GET", status_code="404", region="us-west"}`
- `http_requests_total{method="POST", status_code="200", region="eu-west"}`

Each of these represents a unique time series identified by the combination of the metric name and its labels.

### 2. **Time Series Data Model**
Internally, Prometheus stores the time series data in a **sorted series** based on:
- **Metric name**
- **Label set**

Each time series is stored as a sequence of **timestamp-value pairs** (known as samples). Each sample consists of:
- A **timestamp**: When the sample was recorded.
- A **value**: The value for that particular timestamp.

For example, for a time series `http_requests_total{method="GET", status_code="200", region="us-west"}`, Prometheus would store data like:
```
http_requests_total{method="GET", status_code="200", region="us-west"} 1 1609459200
http_requests_total{method="GET", status_code="200", region="us-west"} 2 1609462800
http_requests_total{method="GET", status_code="200", region="us-west"} 3 1609466400
```

Where:
- `1`, `2`, `3` are the values (i.e., the number of requests).
- `1609459200`, `1609462800`, `1609466400` are the timestamps.

### 3. **Cardinality in Prometheus**
Cardinality refers to the number of unique time series Prometheus is tracking for a given metric, and it depends on the **number of unique combinations of labels**.

- If you have a metric with **3 labels** (`method`, `status_code`, `region`) and each label has multiple possible values, the cardinality of that metric increases by the number of possible combinations of those label values.
- For example, if you have:
  - 4 possible values for `method`
  - 3 possible values for `status_code`
  - 2 possible values for `region`

Then the cardinality for the metric `http_requests_total` would be:
```
4 (method) × 3 (status_code) × 2 (region) = 24 unique time series
```

### 4. **How Prometheus Stores Data (On Disk)**
Prometheus uses a **chunk-based storage system** to store time series data on disk. Here's how it stores the data:

1. **Block-Based Storage**: Prometheus stores time series data in **blocks** on disk. Each block is a directory that contains time series data for a specific time range (e.g., a week or a few days).
   
   Each block contains:
   - A list of time series (identified by their metric name and labels).
   - For each time series, it stores the samples (timestamps and values).
   
   Prometheus uses **compression** (typically Snappy compression) to reduce storage overhead.

2. **Indexing**: Prometheus keeps an index to quickly retrieve data by metric name and labels. The index allows Prometheus to map a combination of label values to the appropriate time series.

   - This index is crucial for efficient querying, but it also contributes to the cardinality issue because the more unique combinations of labels, the larger the index becomes.
   - The index is stored in a format called **TSDB index file**.

### 5. **Chunk Storage**
For each time series, Prometheus stores its data in chunks, where each chunk contains samples for a given period (e.g., 1 hour). The chunk file contains:
- **Timestamps**: When the data was recorded.
- **Values**: The values associated with those timestamps.
- **Tombstones**: Markers for deleted time series (if applicable).

Prometheus uses a **time-ordered** system for chunking, meaning that the samples are ordered by time within each chunk. This is done to ensure efficient storage and retrieval.

### 6. **Handling High Cardinality**
When you have high cardinality (many unique combinations of labels), Prometheus can face issues with:
- **Increased disk usage**: More unique time series means more data to store.
- **Performance degradation**: Querying and scraping metrics with high cardinality can slow down due to the large number of time series Prometheus must manage.

To mitigate high cardinality issues, it’s essential to:
- Avoid using high-cardinality labels (e.g., unique user IDs or session IDs) unless absolutely necessary.
- Use **aggregation** in queries to reduce the number of unique time series queried.
- **Sharding**: In some cases, you might need to run multiple Prometheus instances to split the load.

### 7. **Prometheus Internal Data Flow**
Here’s a rough flow of how Prometheus handles data:
1. **Scraping**: Prometheus scrapes metrics from endpoints (e.g., HTTP servers) and pulls the time series data.
2. **Data Processing**: The scraped data is processed, and time series with the corresponding metric name and labels are created.
3. **Storage**: Time series data is stored in the disk as chunks, with an index for fast retrieval.
4. **Querying**: When a query is made, Prometheus retrieves the appropriate time series from storage, processes it, and returns the results.

### Conclusion
Prometheus stores time series data using a **metric name** and **labels** combination, with each unique combination of labels resulting in a unique time series. The data is stored in chunks, compressed to save space, and indexed for efficient querying. However, as the cardinality (number of unique combinations of labels) increases, the system’s resource requirements (disk space, memory, CPU, etc.) also increase. High cardinality can lead to performance issues, which is why careful consideration of label values and aggregation is essential when using Prometheus at scale.


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

# How to Mitigate Cardinality Issues with example

To mitigate cardinality issues in Prometheus, we can adopt various strategies such as limiting the number of labels, reducing granularity, using relabeling, and adjusting scrape intervals. Let’s walk through a simple example of how to **reduce cardinality** by **aggregating metrics** and using **relabeling**.

### Use Case Example:

Imagine you are monitoring HTTP request counts for a web application with the metric `http_requests_total` that includes the following labels:

- `method`: (GET, POST, PUT)
- `status_code`: (200, 404, 500)
- `endpoint`: (login, checkout, home, profile, etc.)

You might find that over time, the number of unique time series grows because of the large number of endpoints. As a result, you end up with a high cardinality problem. To reduce cardinality, we can aggregate the `endpoint` label into higher-level categories (e.g., "user-related", "checkout-related") and relabel metrics accordingly.

### Step 1: Use Label Aggregation

Instead of tracking each individual endpoint (`login`, `checkout`, `home`, etc.), we can aggregate them into broader categories, such as:

- `user_endpoints` (e.g., `login`, `profile`)
- `commerce_endpoints` (e.g., `checkout`, `cart`)

### Step 2: Example Metric with High Cardinality

#### Before Aggregation:
```yaml
http_requests_total{method="GET", status_code="200", endpoint="login"}
http_requests_total{method="GET", status_code="404", endpoint="login"}
http_requests_total{method="POST", status_code="500", endpoint="checkout"}
# etc.
```

For each combination of `method`, `status_code`, and `endpoint`, Prometheus will create a separate time series, which can grow rapidly.

#### After Aggregation:
We aggregate the endpoints into broader categories, such as "user_endpoints" and "commerce_endpoints":

```yaml
http_requests_total{method="GET", status_code="200", endpoint="user_endpoints"}
http_requests_total{method="GET", status_code="404", endpoint="user_endpoints"}
http_requests_total{method="POST", status_code="500", endpoint="commerce_endpoints"}
# etc.
```

This reduces the number of unique time series because we are now tracking only 2 types of endpoints rather than every single one.

### Step 3: Relabeling in Prometheus Configuration

Prometheus provides a powerful feature called **relabeling** to modify or reduce labels before scraping the data.

Here’s an example of how you might configure Prometheus to relabel and aggregate endpoints:

#### Prometheus Configuration (prometheus.yml)

```yaml
scrape_configs:
  - job_name: 'web_application'
    static_configs:
      - targets: ['localhost:8080']  # Target your application
    relabel_configs:
      - source_labels: [__name__, endpoint]
        target_label: endpoint
        replacement: "user_endpoints"  # Aggregate login, profile into user_endpoints
        action: replace
      - source_labels: [__name__, endpoint]
        target_label: endpoint
        replacement: "commerce_endpoints"  # Aggregate checkout, cart into commerce_endpoints
        action: replace
```

In this example, we use **`relabel_configs`** to change the value of the `endpoint` label for different types of endpoints. The `relabel_configs` feature allows us to map many individual `endpoint` values to broader categories. 

#### How it works:
- For every request to the `http_requests_total` metric, Prometheus will change the `endpoint` label from values like `login`, `profile`, `checkout`, etc., to either `user_endpoints` or `commerce_endpoints` depending on the `endpoint` value.
- This way, we’re reducing the number of unique time series tracked by Prometheus, as it now tracks only two aggregate categories instead of tracking every single endpoint.

### Step 4: Adjusting Scrape Interval

Another way to reduce load and manage high cardinality is to adjust the **scrape interval**. By reducing the frequency of scraping (for less critical metrics), you can reduce the amount of data Prometheus needs to handle.

For example:

```yaml
scrape_configs:
  - job_name: 'web_application'
    scrape_interval: 30s  # Scrape every 30 seconds instead of 15 seconds for less critical metrics
    static_configs:
      - targets: ['localhost:8080']
```

This reduces the number of data points Prometheus needs to scrape, thus helping reduce memory and processing overhead.

### Step 5: Query Optimization

When querying Prometheus, use **aggregation operators** like `sum()`, `avg()`, or `rate()` to reduce the complexity of queries over many time series. For instance:

Instead of querying all individual endpoints:

```prometheus
sum(http_requests_total{status_code="200", endpoint="user_endpoints"})
```

This query will give you the total count of `200` status code requests for user-related endpoints (e.g., `login` and `profile`) combined, without having to query each individual time series.

### Benefits of This Approach:

1. **Reduced Cardinality**:
   - By aggregating `endpoint` values into broader categories (`user_endpoints`, `commerce_endpoints`), you reduce the number of time series that Prometheus needs to store and query.
   
2. **Lower Memory Usage**:
   - With fewer time series, Prometheus will use less memory, which leads to better performance and less strain on the system.
   
3. **Improved Query Performance**:
   - Fewer time series to query means faster query performance and reduced query complexity.

4. **Easier Management**:
   - Having fewer time series to manage makes it easier to maintain and scale your Prometheus setup.

### Summary:

To mitigate cardinality issues in Prometheus:
1. **Aggregate labels**: Combine similar or related labels into broader categories to reduce the number of unique time series.
2. **Relabel metrics**: Use `relabel_configs` in Prometheus to transform and combine metrics dynamically during scraping.
3. **Adjust scrape interval**: For non-critical metrics, adjust the scrape interval to reduce overhead.
4. **Query optimization**: Use aggregation functions to work with fewer time series.

By following these steps, you can keep your Prometheus setup efficient and manageable while monitoring a complex web application.

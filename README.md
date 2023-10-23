![Python-Rust](https://github.com/nogibjj/IDS706-Mini-Project-8-sp699/actions/workflows/python-rust.yml/badge.svg)
# IDS-706-Data-Engineering :computer:

## Mini Project 8 :page_facing_up: 

## :ballot_box_with_check: Requirements
* Take an existing Python script for data processing</br>
* Rewrite it in Rust</br>
* Highlight improvements in speed and resource usage

## :ballot_box_with_check: To-do List
* __Learn how to work with Rust__: To understand how to rewrite Python script in Rust and figure out why Rust is used.</br>

## :ballot_box_with_check: Dataset
`births14.csv`
  - Every year, the US releases to the public a large data set containing information on births recorded in the country. This data set has been of interest to medical researchers who are studying the relation between habits and practices of expectant mothers and the birth of their children. This is a random sample of 1,000 cases from the data set released in 2014.
  - A data frame with 1,000 observations on the following 13 variables.</br>
  - The whole dataset is described in [here](https://www.openintro.org/data/index.php?data=births14).
  - [births14.csv](https://github.com/nogibjj/IDS706-Mini-Project-8-sp699/blob/main/births14.csv)

## :ballot_box_with_check: Main progress
#### Set up the environment to build the code in both Python and Rust.
__`Rust`__
  - Create `main.rs` and `lib.rs` files in the `src` directory for Rust to execute specific functions, __calculating the average weight, measuring data processing time, and monitoring memory usage__.
  - `main.rs`
    ```Rust
    use std::time::Instant;
    use psutil::process::Process;
    use csv::ReaderBuilder;
    use std::error::Error;
    use std::fs::File;

    fn calculate_time_memory(path: &str) -> (i64, f64) {
        // Record the start time
        let start_time = Instant::now();

        // Measure the initial resource usage
        let process = Process::new(std::process::id());
        let start_mem_usage = process.as_ref().expect("Failed to create process").memory_info().expect("Failed to get memory info").rss();

        // Calculate the average
        let _average_result = average(path);
    
        // Record the end time
        let end_time = Instant::now();
    
        // Measure the final resource usage
        let end_mem_usage = process.as_ref().expect("Failed to create process").memory_info().expect("Failed to get memory info").rss();
    
        // Calculate the elapsed time
        let elapsed_time = end_time.duration_since(start_time).as_secs_f64();
    
        println!("Rust-Elapsed Time: {:.7} seconds", elapsed_time);
        println!("Rust-Memory Usage Change: {} kilobytes", end_mem_usage - start_mem_usage);
    
        (end_mem_usage as i64, elapsed_time)
    }
    
    fn average(path: &str) -> Result<f64, Box<dyn Error>> {
        let file = File::open(path)?;
        let mut rdr = ReaderBuilder::new().delimiter(b',').from_reader(file);
    
        let mut sum = 0.0;
        let mut count = 0;
    
        for result in rdr.records() {
            let record = result?;
            if let Some(weight) = record.get(7) {
                if let Ok(parsed_weight) = weight.parse::<f64>() {
                    sum += parsed_weight;
                    count += 1;
                }
            }
        }
    
        if count == 0 {
            return Err("No data found".into());
        }
    
        Ok(sum / count as f64)
    }
    
    fn main() {
        let path = "/workspaces/IDS706-Mini-Project-8-sp699/births14.csv"; // Updated with the actual CSV file path
    
        match average(path) {
            Ok(avg) => println!("Average weight: {:.5}", avg),
            Err(err) => eprintln!("Error: {}", err),
        }
    
        let (end_mem_usage, elapsed_time) = calculate_time_memory(path);
    
        println!("Final Memory Usage: {} kilobytes", end_mem_usage);
        println!("Total Elapsed Time: {:.7} seconds", elapsed_time);
    }
    ```
  - Create a `Cargo.toml` file to execute the Rust files and install the necessary dependencies.</br>
  - `Cargo.toml` </br>
    ```
    [package]
    name = "rust-time-memory"
    version = "0.1.0"
    edition = "2021"
    
    # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
    
    [dependencies]
    rand = "0.8.5"
    time = "0.3.30"
    csv = "1.3.0"
    psutil = "3.2.2"
    serde = "1.0.189"
    serde_derive = "1.0.189"
    ```

__`Python`__
  - Create `__init__.py` and `average.py` files in the `library` directory for Python to execute specific functions, __calculating the average weight, measuring data processing time, and monitoring memory usage__.
  - `average.py`
    ```Python
    import pandas as pd
    import time
    import resource
    
    def average(path):
        birth_data = pd.read_csv(path)
    
        weight_avg = birth_data["weight"].mean()
        return weight_avg
    
    def calculate_time_memory(path):
        # Record the start time
        start_time = time.time()
    
        # Measure the initial resource usage
        start_mem_usage = resource.getrusage(resource.RUSAGE_SELF).ru_maxrss
    
        # Calculate the average
        average(path)
    
        # Record the end time
        end_time = time.time()
    
        # Calculate the elapsed time
        elapsed_time = end_time - start_time
    
        # Measure the final resource usage
        end_mem_usage = resource.getrusage(resource.RUSAGE_SELF).ru_maxrss
    
        print(f"Python-Elapsed Time: {elapsed_time:.7f} seconds")
        print(f"Python-Memory Usage Change: {end_mem_usage - start_mem_usage} kilobytes")
    
        return end_mem_usage, elapsed_time
    ```
  - Create `main.py` and `test_main.py` to ensure that `average.py` runs correctly and to perform testing.
  - `main.py`
    ```Python
    from library.average import average, calculate_time_memory
    
    data_path = "births14.csv"
    
    if __name__ == "__main__":
        data_path = "births14.csv"  # Updated with the actual CSV file path
    
        avg_weight = average(data_path)
        print(f"Average weight: {avg_weight:.5}")
    
        end_mem_usage, elapsed_time = calculate_time_memory(data_path)
    
        print(f"Final Memory Usage: {end_mem_usage} kilobytes")
        print(f"Total Elapsed Time: {elapsed_time:.7} seconds")
    ```
  - `test_main.py`
    ```Python
    from library.average import average, calculate_time_memory

    data_path = "births14.csv"
    
    def test_average():
        result = average("births14.csv")
        expected_result = 7.19816
    
        assert result == expected_result, "Test has failed."
    
    def test_calculate_time_memory():
        result = calculate_time_memory("births14.csv")
        
        assert result is not None, "Test has failed."
    
    if __name__ == "__main__":
        test_average()
        test_calculate_time_memory()
    ```

## :ballot_box_with_check: Compare the results

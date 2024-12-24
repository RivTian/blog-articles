#Rust 

In this article, we will conduct a performance comparison of three of the most popular Rust web frameworks: Axum, Actix, and Rocket.

### How we are going to test

On each framework we are going to implement simple Web-service with three endpoints:

| POST /test/simple | Receives a param in JSON request, formats it and responds with JSON                                         |
| ----------------- | ----------------------------------------------------------------------------------------------------------- |
| POST /test/timed  | Receives a param in JSON request, sleeps 20 ms, then formats it as previous endpoint and responds with JSON |
| POST /test/bcrypt | Receives a param in JSON request, hashes it using Bcrypt with cost=10 and responds with JSON                |

The first endpoint allows us to measure the pure overhead of the framework. The second simulates a light database query or an external API call, while the third simulates some heavy business logic. All endpoints accept and respond with a JSON object containing a single string field named "payload".

The code for all frameworks is written following tutorials from their respective official websites, and all performance-related configurations are kept at their default values.

### Axum

Announced on July 30, 2021, Axum is the youngest framework in our review and simultaneously the most popular. It is developed by the same team responsible for Tokio, the most popular asynchronous runtime for Rust, also used as the foundation in Actix and Rocket.

One of the key advantages of this framework is the ability to declare endpoints without macros. This feature enhances code readability, makes compiler messages more understandable, and provides better IDE syntax highlighting and auto-completion. Other advantages according to authors:

- Declaratively parse requests using extractors.
- Simple and predictable error handling model.
- Generate responses with minimal boilerplate.
- Take full advantage of the tower and tower-http ecosystem of middleware, services, and utilities.

Documentation quality is very good - I have no issues with writing my test code.

Main application function is a normal async tokio main function, so you can do async initizalization.

GitHub: [https://github.com/tokio-rs/axum](https://github.com/tokio-rs/axum)  
Documentation: [https://docs.rs/axum/latest/axum/](https://docs.rs/axum/latest/axum/)  
Total downloads on [crates.io](https://crates.io/crates/axum): **23M**

#### Code

main.rs

```rust
use std::str::FromStr;
use std::time::Duration;
use axum::Json;
use axum::response::IntoResponse;
use tokio::time::sleep;

#[derive(Debug, serde::Serialize, serde::Deserialize)]
struct Data {
	payload: String
}

async fn simple_endpoint(Json(param): Json<Data>) -> impl IntoResponse {
	Json(Data {
		payload: format!("Hello, {}", param.payload)
	})
}

async fn timed_endpoint(Json(param): Json<Data>) -> impl IntoResponse {
	sleep(Duration::from_millis(20)).await;
	Json(Data {
		payload: format!("Hello, {}", param.payload)
	})
}

async fn bcrypt_endpoint(Json(param): Json<Data>) -> impl IntoResponse {
	Json(Data {
		payload: bcrypt::hash(&param.payload, 10).unwrap()
	})
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
	env_logger::init_from_env(env_logger::Env::default().default_filter_or("info"));
	let router = axum::Router::new()
		.route("/test/simple", axum::routing::post(simple_endpoint))
		.route("/test/timed", axum::routing::post(timed_endpoint))
		.route("/test/bcrypt", axum::routing::post(bcrypt_endpoint));
	let address = "0.0.0.0";
	let port = 3000;
	log::info!("Listening on http://{}:{}/", address, port);
	axum::Server::bind(
		&std::net::SocketAddr::new(
			std::net::IpAddr::from_str(&address).unwrap(),
			port
		)
	).serve(router.into_make_service()).await?;
	Ok(())
}
```

Cargo.toml

```toml
[package]
name = "rust_web_benchmark"
version = "0.1.0"
edition = "2021"

[dependencies]
log = "0.4.20"
env_logger = "0.10.0"
tokio = { version = "1", features = ["macros", "rt-multi-thread"] }
axum = "0.6.20"
serde = { version = "1.0.189", features = ["derive"] }
bcrypt = "0.15.0"
```

### Actix

The first release on GitHub is dated October 31, 2017.

Key advantages according to authors:

- Type safe
- Feature rich (HTTP/2, logging etc)
- Extensible
- Blazing fast

Rocket uses macros for endpoint descriptions. The main application function is compatible with a typical Tokio async main function, allowing for asynchronous initialization. The documentation for beginners is excellent, as I had no difficulties writing my test code despite having no prior experience with Actix.

Official site: [https://actix.rs/](https://actix.rs/)  
Total downloads on [crates.io](https://crates.io/crates/actix): **5.8M**

#### Code

main.rs

```rust
use std::time::Duration;
use actix_web::{post, App, HttpResponse, HttpServer, Responder};
use actix_web::web::Json;
use tokio::time::sleep;

#[derive(Debug, serde::Serialize, serde::Deserialize)]
struct Data {
	payload: String
}

#[post("/test/simple")]
async fn simple_endpoint(Json(param): Json<Data>) -> impl Responder {
	HttpResponse::Ok().json(Json(Data {
		payload: format!("Hello, {}", param.payload)
	}))
}

#[post("/test/timed")]
async fn timed_endpoint(Json(param): Json<Data>) -> impl Responder {
	sleep(Duration::from_millis(20)).await;
    HttpResponse::Ok().json(Json(Data {
		payload: format!("Hello, {}", param.payload)
	}))
}

#[post("/test/bcrypt")]
async fn bcrypt_endpoint(Json(param): Json<Data>) -> impl Responder {
	HttpResponse::Ok().json(Json(Data {
		payload: bcrypt::hash(&param.payload, 10).unwrap()
	}))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
	env_logger::init_from_env(env_logger::Env::default().default_filter_or("info"));
	let address = "0.0.0.0";
	let port = 3000;
	log::info!("Listening on http://{}:{}/", address, port);
    HttpServer::new(|| {
        App::new()
            .service(simple_endpoint)
            .service(timed_endpoint)
			.service(bcrypt_endpoint)
    })
        .bind((address, port))?
        .run()
        .await
}
```

Cargo.toml

```toml
[package]
name = "rust_web_benchmark"
version = "0.1.0"
edition = "2021"

[dependencies]
log = "0.4.20"
env_logger = "0.10.0"
tokio = { version = "1", features = ["macros", "rt-multi-thread"] }
actix-web = "4"
serde = { version = "1.0.189", features = ["derive"] }
bcrypt = "0.15.0"
```

### Rocket

Released in 2016, Rocket is the oldest framework in this review. Prior to version 0.5, it utilized its own asynchronous runtime. However, at version 0.5, Rocket migrated to using Tokio as its async runtime.

Key advantages according to authors:

- Type safe
- Boilerplate free
- Easy to use
- Extensible

Macros are actively used for defining handlers and endpoints in Rocket. It also employs its own **rocket::launch** macro for declaring the main function, which is not compatible with the standard Tokio main function. The main function returns a router instance that is internally started by Rocket itself.

Despite the declaration of compatibility with Rust stable in version 0.5, I was unable to compile it without Rust Nightly due to a dependency on Pear. Thus, this test is the only one that compiled using Rust Nightly.

Furthermore, there have been significant changes in documentation between versions 0.4 and 0.5 due to breaking API changes. This can cause confusion when searching for solutions on Google because some code examples for 0.4 won't work for 0.5. I spent much more time writing test code with Rocket than with Axum and Actix because I had to troubleshoot errors that arose after copying and pasting examples from the documentation. While experienced Rocket users may not find this problematic, it poses a significant challenge for beginners.

Official site: [https://rocket.rs/](https://rocket.rs/)  
Total downloads on [crates.io](https://crates.io/crates/rocket): **3.7M**

#### Code

main.rs

```rust
use std::time::Duration;
use tokio::time::sleep;
use rocket::serde::json::Json;

#[macro_use] extern crate rocket;

#[derive(Debug, serde::Serialize, serde::Deserialize)]
struct Data {
	payload: String
}

#[post("/test/simple", data = "<param>")]
async fn simple_endpoint(param: Json<Data>) -> Json<Data> {
	Json(Data {
		payload: format!("Hello, {}", param.into_inner().payload)
	})
}

#[post("/test/timed", data = "<param>")]
async fn timed_endpoint(param: Json<Data>) -> Json<Data> {
	sleep(Duration::from_millis(20)).await;
	Json(Data {
		payload: format!("Hello, {}", param.into_inner().payload)
	})
}

#[post("/test/bcrypt", data = "<param>")]
async fn bcrypt_endpoint(param: Json<Data>) -> Json<Data> {
	Json(Data {
		payload: bcrypt::hash(&param.into_inner().payload, 10).unwrap()
	})
}

#[launch]
fn rocket() -> _ {
	rocket::build()
		.configure(rocket::Config::figment()
			.merge(("address", "0.0.0.0"))
			.merge(("port", 3000))
		)
		.mount("/", routes![
			simple_endpoint,
			timed_endpoint,
			bcrypt_endpoint
		])
}
```

Cargo.toml

```toml
[package]
name = "rust_web_benchmark"
version = "0.1.0"
edition = "2021"

[dependencies]
tokio = "1"
rocket = { version = "0.5.0-rc.3", features = ["json"] }
serde = { version = "1.0.189", features = ["derive"] }
bcrypt = "0.15.0"
```

### The benchmark

To test the performance of all test services, we wrote a small Rust program using Tokio and Reqwest. This program spawns N Tokio tasks, with each task executing M requests to a specified endpoint using a predefined JSON string as the payload. A request is considered successful if it returns a status of 200 OK. The program measures the time for each successful request and prints the average and median times at the end. It also keeps a count of failed requests, although I anticipate having zero failed requests.

Additionally, the program calculates and prints the Requests Per Second (RPS) by dividing the count of successful requests by the total time elapsed from the launch of the first task to the termination of the last task.

#### Code

main.rs

```rust
use reqwest::StatusCode;

static REQ_PAYLOAD: &str = "{\n\t\"payload\": \"world\"\n}\n";

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
	let args = std::env::args().collect::<Vec<_>>();
	if args.len() != 4 {
		println!("Usage: {} url thread-count request-count", args[0]);
		return Ok(());
	}
	
	let url = args[1].clone();
	let request_count = args[2].parse().unwrap();
	let thread_count = args[3].parse().unwrap();
	
	let client = reqwest::Client::new();
	let start = std::time::Instant::now();
	let handles = (0..thread_count).map(|_| {
		let url = url.clone();
		let client = client.clone();
		tokio::spawn(async move {
			let mut error_count = 0;
			let mut results = Vec::new();
			for _ in 0..request_count {
				let start = std::time::Instant::now();
				let res = client
					.post(&url)
					.header("Content-Type", "application/json")
					.body(REQ_PAYLOAD)
					.send()
					.await
					.unwrap();
				if res.status() == StatusCode::OK {
					res.text().await.unwrap();
					let elapsed = std::time::Instant::now().duration_since(start);
					results.push(elapsed.as_micros());
				} else {
					error_count += 1;
				}
			}
			(results, error_count)
		})
	}).collect::<Vec<_>>();
	
	let mut results = Vec::new();
	let mut error_count = 0;
	for handle in handles {
		let (out, err_count) = handle.await.unwrap();
		results.extend(out.into_iter());
		error_count += err_count;
	}
	let elapsed = std::time::Instant::now().duration_since(start);
	let rps = results.len() as f64 / elapsed.as_secs_f64();
	
	results.sort();
	println!(
		"average={}us, median={}us, errors={}, total={}, rps={}",
		results.iter()
			.copied()
			.reduce(|a, b| a + b).unwrap() / results.len() as u128,
		results[results.len() / 2],
		error_count,
		results.len(),
		rps
	);
	Ok(())
}
```

Cargo.toml

```toml
[package]
name = "bench"
version = "0.1.0"
edition = "2021"

[dependencies]
tokio = { version = "1", features = ["macros", "rt-multi-thread"] }
reqwest = "0.11.22"
```

### Results

Each service was launched in a separate Docker container for convenient resource consumption measurement. Subsequently, the same endpoint in each container was benchmarked one by one. Afterward, all containers were restarted, and I proceeded to the next endpoint. This entire sequence was repeated three times, each with a different container order to avoid the possibility of the last benchmarked service experiencing CPU throttling or the first service getting stuck in CPU powersave mode. The results were then averaged.

For simple and timed tests, I utilized 100 tasks and 100 requests. In the case of the bcrypt test, I employed 10 tasks and 50 requests.

| Test            | Parameter        | **Axum** | **Actix** | **Rocket** |
| --------------- | ---------------- | -------- | --------- | ---------- |
| Simple          | **Average (ms)** | 7.727    | 7.239     | 12.971     |
| **Median (ms)** | 3.698            | 3.1      | 9.097     |            |
| **RPS**         | 12010            | 12483    | 7419      |            |
| Timed           | **Average (ms)** | 25.922   | 25.764    | 26.402     |
| **Median (ms)** | 22.379           | 21.906   | 22.659    |            |
| **RPS**         | 3799             | 3789     | 3696      |            |
| Bcrypt          | **Average (ms)** | 493      | 505       | 501        |
| **Median (ms)** | 474              | 486      | 503       |            |
| **RPS**         | 93               | 86       | 91        |            |

As you can see, Axum and Actix exhibit very similar response times, while Rocket performs less optimally. Moreover, in real applications, the difference might be less pronounced, particularly when considering timed and bcrypt endpoints, which simulate real business logic rather than merely benchmarking the framework's overhead.

| RAM usage                | **Axum** | **Actix** | **Rocket** |
| ------------------------ | -------- | --------- | ---------- |
| **Startup**              | 0.75 MiB | 1.3 MiB   | 0.97 MiB   |
| **While test (maximum)** | 71 MiB   | 71 MiB    | 102 MiB    |
| **After test**           | 0.91 MiB | 2.4 MiB   | 1.8 MiB    |

Axum has the best memory consumption, both in standby and under load. Actix matches Axum's performance under load but has the worst standby memory consumption. Rocket, on the other hand, exhibits average standby memory consumption but performs worse under load.

### Conclusions

My personal choice is Axum because it offers performance very close to Actix, the best memory consumption, and macro-free route declarations. It also boasts excellent documentation and a large community. Any minor differences in response times between Axum and Actix are likely to be eliminated in future releases. Actix takes my second spot for its strong performance and good documentation. I don't see any advantages for Rocket these days. Perhaps it was the best framework at the time of its initial release, but now it offers no discernible benefits, and the documentation is in disarray after the 0.4 → 0.5 breaking changes.

[Benchmark source code on GitHub](https://github.com/KivApple/rust_web_benchmark)
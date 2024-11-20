

1、 fastwebsocket

```rust
use std::collections::HashMap;
use std::io::Read;
use std::net::SocketAddr;
use std::sync::Arc;

use deno_core::futures::channel::mpsc;
use deno_core::futures::channel::mpsc::UnboundedReceiver;
use deno_core::futures::channel::mpsc::UnboundedSender;
use deno_core::futures::select;
use fastwebsockets::upgrade;
use fastwebsockets::Frame;
use fastwebsockets::OpCode;
use fastwebsockets::WebSocketError;
use http_body_util::Empty;
use hyper::body::Bytes;
use hyper::body::Incoming;
use hyper::server::conn::http1;
use hyper::service::service_fn;
use hyper::HeaderMap;
use hyper::Request;
use hyper::Response;
use reqwest::Client;
use tokio::net::TcpListener;
// use tokio::sync::mpsc;

use std::fs::{create_dir_all, File};
use std::io::Write;
use std::path::Path;

// has_userScripteSetUp?

#[derive(Debug)]
pub struct InspectorServer {
  pub host: SocketAddr,
  pub inspector_rx: UnboundedReceiver<String>,
  pub inspector_tx: UnboundedSender<String>,
}

impl InspectorServer {
  pub fn new(host: SocketAddr) -> InspectorServer {
    // let ws_listener = TcpListener::bind(host).await.unwrap();
    let (inspector_tx, inspector_rx) = mpsc::unbounded::<String>();

    InspectorServer {
      host,
      inspector_rx,
      inspector_tx,
    }
  }
}

pub async fn handle_ws(stream: tokio::net::TcpStream) {
  let io = hyper_util::rt::TokioIo::new(stream);
  let conn_fut = http1::Builder::new()
    .serve_connection(io, service_fn(server_upgrade))
    .with_upgrades();
  if let Err(e) = conn_fut.await {
    println!("An error occurred: {:?}", e);
  }
}

fn write_js_to_file(
  js_code: String,
  file_path: &str,
) -> Result<(), std::io::Error> {
  // 确保目录存在
  if let Some(parent_dir) = Path::new(file_path).parent() {
    create_dir_all(parent_dir)?;
  }

  // 创建或打开文件
  let mut file = File::create(file_path)?;

  // 写入 JavaScript 代码
  file.write_all(js_code.as_bytes())?;

  Ok(())
}

async fn handle_client(fut: upgrade::UpgradeFut) -> Result<(), WebSocketError> {
  let js_code = r#"
  export default {
    async fetch(event, context) {
      return new Response("hello world from debugger");
    }
  }
  "#
  .to_string();

  let file_path = "conf/mochen.js";
  match write_js_to_file(js_code, file_path) {
    Ok(_) => println!("JavaScript 代码已成功写入 {}", file_path),
    Err(e) => println!("写入文件时出错: {}", e),
  }

  let mut ws = fastwebsockets::FragmentCollector::new(fut.await?);
  let client = Client::new();
  loop {
    let frame = ws.read_frame().await?;
    match frame.opcode {
      OpCode::Close => break,
      OpCode::Text | OpCode::Binary => {
        // 提取并打印帧的 opcode 和 payload
        let opcode = frame.opcode;

        // 获取 payload 的引用
        let payload: &[u8] = &frame.payload;

        // 打印 opcode 和 payload 的长度
        println!(
          "Received frame: opcode = {:?}, payload length = {}",
          opcode,
          payload.len()
        );

        // 如果是文本帧，尝试转换为字符串并打印
        // if let OpCode::Text = opcode {
        let mut headers = HeaderMap::new();
        headers.insert("Accept", "*/*".parse().unwrap());
        headers.insert("Host", "a.com".parse().unwrap());
        headers.insert("x-er-id", "mochen.subdomain-kksnkq".parse().unwrap());
        headers.insert("x-er-inspector", "mochen".parse().unwrap());
        headers.insert("x-er-context", "eyJzaXRlX2lkIjogIjYyMjcxODQ0NjgwNjA4IiwgInNpdGVfbmFtZSI6ICJjb21wdXRlbHguYWxpY2RuLXRlc3QuY29tIiwgInNpdGVfcmVjb3JkIjogIm1vY2hlbi1uY2RuLmNvbXB1dGVseC5hbGljZG4tdGVzdC5jb20iLCAiYWxpdWlkIjogIjEzMjI0OTI2ODY2NjU2MDgiLCAic2NoZW1lIjoiaHR0cCIsICAiaW1hZ2VfZW5hYmxlIjogdHJ1ZX0=".parse().unwrap());

        if let Ok(text) = String::from_utf8(payload.to_vec()) {
          let response = client
            .get("http://localhost:18080/abcd")
            .headers(headers)
            .body(text.clone()) // 将收到的文本作为请求体
            .send()
            .await;

          let response_frame = match response {
            Ok(res) => {
              let status = res.status().to_string();
              let body = res
                .text()
                .await
                .unwrap_or_else(|_| "Failed to read response body".to_string());
              let response_text =
                format!("HTTP Status: {}, Body: {}", status, body);

              // println!(
              //   "HTTP response: {:?}, {:?}",
              //   res.status(),
              //   res.text().await.unwrap()
              // )
              Frame::new(
                true,
                OpCode::Text,
                None,
                response_text.into_bytes().into(),
              )
            }
            Err(err) => {
              // eprintln!("Error sending HTTP request: {}", err),
              let error_message =
                format!("Error sending HTTP request: {}", err);
              Frame::new(
                true,
                OpCode::Text,
                None,
                error_message.into_bytes().into(), // 将错误信息作为负载
              )
            }
          };

          ws.write_frame(response_frame).await?;

          println!("Received text: {}", text);
        } else {
          println!("Received non-UTF8 text frame");
        }
        // }

        ws.write_frame(frame).await?;
      }
      _ => {}
    }
  }

  Ok(())
}
async fn server_upgrade(
  mut req: Request<Incoming>,
) -> Result<Response<Empty<Bytes>>, WebSocketError> {
  let (response, fut) = upgrade::upgrade(&mut req)?;

  tokio::task::spawn(async move {
    if let Err(e) = tokio::task::unconstrained(handle_client(fut)).await {
      eprintln!("Error in websocket connection: {}", e);
    }
  });

  Ok(response)
}

```


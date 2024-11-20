

# 参数读取

```RUST
  let url = env::args()
    .nth(1)
    .unwrap_or_else(|| panic!("this program requires at least one argument"));
```

## 标准输入输出

```rust
async fn read_stdin2(tx: futures_channel::mpsc::UnboundedSender<Message>) {
  let mut stdin = tokio::io::stdin();
  loop {
    let mut buf = vec![0; 1024];
    let n = match stdin.read(&mut buf).await {
      Err(_) | Ok(0) => break,
      Ok(n) => n,
    };
    buf.truncate(n);
    tx.unbounded_send(Message::binary(buf)).unwrap();
  }
}

async fn read_stdin1(tx: futures_channel::mpsc::UnboundedSender<Message>) {
  let stdin = tokio::io::stdin();
  let reader = BufReader::new(stdin);
  let mut lines = reader.lines();

  while let Some(line) = lines.next_line().await.unwrap() {
    let message = json!({
      "name": "setUserScript",
      "data": line
    });
    let json_message = serde_json::to_string(&message).unwrap();
    tx.unbounded_send(Message::text(json_message)).unwrap();
  }
}
```




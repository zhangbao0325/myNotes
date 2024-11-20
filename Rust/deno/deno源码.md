

# primordials

  ArrayBufferIsView,
  ArrayBufferPrototype,
  ArrayBufferPrototypeGetByteLength,
  ArrayBufferPrototypeSlice,
  ArrayPrototypeEvery,
  ArrayPrototypeFilter,
  ArrayPrototypeFind,
  ArrayPrototypeIncludes,
  DataViewPrototypeGetBuffer,
  DataViewPrototypeGetByteLength,
  DataViewPrototypeGetByteOffset,
  JSONParse,
  JSONStringify,
  MathCeil,
  ObjectAssign,
  ObjectHasOwn,
  ObjectPrototypeIsPrototypeOf,

  ArrayPrototypePush,

  ArrayPrototypeSplice,

  ArrayPrototypeFilter,

  ArrayPrototypeIncludes,

  Error,

  ObjectPrototypeIsPrototypeOf,

  Promise,

  PromisePrototypeThen,

  PromisePrototypeCatch,

  SafeArrayIterator,

  String,

  StringPrototypeStartsWith,

  StringPrototypeToLowerCase,

  TypeError,

  TypedArrayPrototypeGetSymbolToStringTag,

  ArrayIsArray,

  ArrayPrototypeSort,

  ArrayPrototypeJoin,

  ObjectFromEntries,

  ObjectHasOwn,

  RegExpPrototypeTest,

  Symbol,

  SymbolFor,

  SymbolIterator,

  StringPrototypeReplaceAll,

  StringPrototypeCharCodeAt,

  ArrayPrototypeSlice,

  MapPrototypeGet,

  MapPrototypeSet,

  MathRandom,

  ObjectFreeze,

  SafeMap,

  SafeRegExp,

  StringFromCharCode,

  StringPrototypeTrim,

  StringPrototypeSlice,

  StringPrototypeSplit,

  StringPrototypeReplace,

  StringPrototypeIndexOf,

  StringPrototypePadStart,

  StringPrototypeCodePointAt,

  TypedArrayPrototypeSubarray,

  Uint8Array,







ReadableStreamPrototype

  SafeArrayIterator,
  SafeWeakMap,
  StringFromCharCode,
  StringPrototypeCharCodeAt,
  StringPrototypeToLowerCase,
  StringPrototypeToUpperCase,
  Symbol,
  SymbolFor,
  SyntaxError,
  TypeError,
  TypedArrayPrototypeGetBuffer,
  TypedArrayPrototypeGetByteLength,
  TypedArrayPrototypeGetByteOffset,
  TypedArrayPrototypeGetSymbolToStringTag,
  TypedArrayPrototypeSlice,
  Uint8Array,
  WeakMapPrototypeGet,
  WeakMapPrototypeSet,

  ArrayBufferPrototype,

  ArrayBufferIsView,

  ArrayPrototypeForEach,

  ArrayPrototypePush,

  ArrayPrototypeSort,

  ArrayIteratorPrototype,

  BigInt,

  BigIntAsIntN,

  BigIntAsUintN,

  DataViewPrototypeGetBuffer,

  Float32Array,

  Float64Array,

  FunctionPrototypeBind,

  Int16Array,

  Int32Array,

  Int8Array,

  isNaN,

  MathFloor,

  MathFround,

  MathMax,

  MathMin,

  MathPow,

  MathRound,

  MathTrunc,

  Number,

  NumberIsFinite,

  NumberIsNaN,

  *// deno-lint-ignore camelcase*

  NumberMAX_SAFE_INTEGER,

  *// deno-lint-ignore camelcase*

  NumberMIN_SAFE_INTEGER,

  ObjectAssign,

  ObjectCreate,

  ObjectDefineProperties,

  ObjectDefineProperty,

  ObjectGetOwnPropertyDescriptor,

  ObjectGetOwnPropertyDescriptors,

  ObjectGetPrototypeOf,

  ObjectHasOwn,

  ObjectPrototypeIsPrototypeOf,

  ObjectIs,

  PromisePrototypeThen,

  PromiseReject,

  PromiseResolve,

  ReflectApply,

  ReflectDefineProperty,

  ReflectGetOwnPropertyDescriptor,

  ReflectHas,

  ReflectOwnKeys,

  RegExpPrototypeTest,

  SafeRegExp,

  SafeSet,

  SetPrototypeEntries,

  SetPrototypeForEach,

  SetPrototypeKeys,

  SetPrototypeValues,

  SetPrototypeHas,

  SetPrototypeClear,

  SetPrototypeDelete,

  SetPrototypeAdd,

  *// TODO(lucacasonato): add SharedArrayBuffer to primordials*

  *// SharedArrayBufferPrototype*

  String,

  StringFromCodePoint,

  StringPrototypeCharCodeAt,

  Symbol,

  SymbolIterator,

  SymbolToStringTag,

  TypedArrayPrototypeGetBuffer,

  TypedArrayPrototypeGetSymbolToStringTag,

  TypeError,

  Uint16Array,

  Uint32Array,

  Uint8Array,

  Uint8ClampedArray,





```js
const {

} = primordials;
```







Init v8 callback

```rust

pub(crate) fn initialize_html_stream_cb<'s>(
  scope: &mut v8::HandleScope<'s>,
  context: v8::Local<'s, v8::Context>,
) {
  let global = context.global(scope);
  let mochen_str = v8_static_strings::new(scope, &v8_static_strings::MOCHEN);

  let maybe_mochen_obj_val = global.get(scope, mochen_str.into());

  // If `Deno.core` is already set up, let's exit early.
  if let Some(mochen_obj_val) = maybe_mochen_obj_val {
    if !mochen_obj_val.is_undefined() {
      return;
    }
  }

  let mochen_obj = v8::Object::new(scope);
  
  let key = v8_static_strings::new(scope, &v8_static_strings::MYFUNC);
  let call_mochen_fn = v8::Function::new(scope, call_mochen).unwrap();

  let name = v8::String::new(scope, "MOCHEN").unwrap();

  call_mochen_fn.set_name(name);

  // mochen_obj.set(scope, key.into(), call_mochen_fn.into());
  
  let call_console_fn = v8::Function::new(scope, call_console).unwrap();
  let call_console_key =
  v8_static_strings::new(scope, &v8_static_strings::CALL_CONSOLE);
  mochen_obj.set(scope, call_console_key.into(), call_console_fn.into());
  
  global.set(scope, key.into(), call_mochen_fn.into());
  global.set(scope, mochen_str.into(), mochen_obj.into());

  let scope = &mut v8::ContextScope::new(scope, context);


  let key = v8::String::new(scope, "magicFn").unwrap();
  let name = v8::String::new(scope, "fooBar").unwrap();
  let tmpl = v8::FunctionTemplate::new(scope, myHtmlRewriterBuilder);
  let func = tmpl.get_function(scope).unwrap();
  func.set_name(name);
  global.set(scope, key.into(), func.into());
  // func.set_class_name(v8::String::new(scope, "HTMLStream2").unwrap());
  
  // 创建构造器模板
  let constructor_template = v8::FunctionTemplate::new(scope, html_stream_constructor_callback);
  constructor_template.set_class_name(v8::String::new(scope, "HTMLStreamInner").unwrap());
  
  constructor_template.read_only_prototype();
  // constructor_template.inherit(parent);
  
  // 也可以在这里设置 prototype 和 instance template
  // constructor_template.prototype_template().set() ...

  // 从模板中获取函数
  let constructor = constructor_template.get_function(scope).unwrap();

  // 设置构造函数为全局对象的属性
  let key = v8::String::new(scope, "HTMLStreamInner").unwrap();
  let global = context.global(scope);
  global.set(scope, key.into(), constructor.into());
}
```

```
fn create_js_function<'a>(
  scope: &mut v8::HandleScope<'a>,
  name: &str,
  callback: impl MapFnTo<v8::FunctionCallback>,
  element_object: &v8::Local<v8::Object>,
) {
  let key = v8::String::new(scope, name).unwrap();
  let template = v8::FunctionTemplate::new(scope, callback);
  let function = template.get_function(scope).unwrap();
  element_object
    .set(scope, key.into(), function.into())
    .unwrap();
}

fn create_element_external<'a>(
  scope: &mut v8::HandleScope<'a>,
  element: &mut Element,
) -> v8::Local<'a, v8::External> {
  let element_ptr = element as *mut Element as *mut c_void;
  v8::External::new(scope, element_ptr)
}
```

```RUST

fn set_tag_name_callback(
  scope: &mut v8::HandleScope,
  args: v8::FunctionCallbackArguments,
  _retval: v8::ReturnValue,
) {
  let tag_name = args.get(0).to_rust_string_lossy(scope);
  // 获取 '__element_data__' 属性，并确认它是一个 External
  let this = args.holder();
  let key = v8::String::new(scope, ELEMENT_DATA).unwrap();
  let external_data = this.get(scope, key.into()).unwrap();
  unsafe {
    let ele_external = v8::Local::<v8::External>::cast(external_data);
    let ele_ptr = ele_external.value();
    let ele = &mut *(ele_ptr as *mut Element);
    ele.set_tag_name(&tag_name);
  }
}

fn before_callback(
  scope: &mut v8::HandleScope,
  args: v8::FunctionCallbackArguments,
  _retval: v8::ReturnValue,
) {
  let content = args.get(0).to_rust_string_lossy(scope);
  let this = args.holder();
  let key = v8::String::new(scope, ELEMENT_DATA).unwrap();
  let external_data = this.get(scope, key.into()).unwrap();
  unsafe {
    let ele_external = v8::Local::<v8::External>::cast(external_data);
    let ele_ptr = ele_external.value();
    let ele = &mut *(ele_ptr as *mut Element);
    ele.before(content.as_str(), ContentType::Html);
  }
}

fn after_callback(
  scope: &mut v8::HandleScope,
  args: v8::FunctionCallbackArguments,
  _retval: v8::ReturnValue,
) {
  let content = args.get(0).to_rust_string_lossy(scope);
  let this = args.holder();
  let key = v8::String::new(scope, ELEMENT_DATA).unwrap();
  let external_data = this.get(scope, key.into()).unwrap();
  unsafe {
    let ele_external = v8::Local::<v8::External>::cast(external_data);
    let ele_ptr = ele_external.value();
    let ele = &mut *(ele_ptr as *mut Element);
    ele.after(content.as_str(), ContentType::Html);
  }
}
```





将Value->Array进行转换：

```rust
fn create_html_rewriter(
  scope: &mut v8::HandleScope<'static>,
  object: &v8::Object,
  handle_cb: v8::Global<v8::Function>,
  receiver: v8::Global<v8::Function>,
) -> Result<HtmlRewriter<'static, Box<dyn FnMut(&[u8])>>, AnyError> {
  let rewriter_type = vec!["element", "comments", "text"];
  let context = scope.get_current_context();
  let global = context.global(scope);
  let handle_cb = RefCell::new(handle_cb);
  let mut element_content_handlers = vec![];

  let scope_ptr = scope as *mut v8::HandleScope;
  // 提取 JavaScript 对象中的元素数组
  for rt in rewriter_type.iter() {
    let name = v8::String::new(scope, rt).unwrap();
    let rewriters_js = object.get(scope, name.into());
    // 检查是否 `Some` 并转换为对象
    if let Some(rewriters_value) = rewriters_js {
      let rewriters_obj = rewriters_value
        .to_object(scope)
        .ok_or_else(|| type_error("Expected rewriters to be an object"))?;

      if !rewriters_obj.is_array() {
        type_error("Expected rewriters to be an array");
      }

      // 进行数组类型转换
      let rewriters_array: v8::Local<v8::Array> = unsafe { std::mem::transmute(rewriters_obj) };

      // 遍历元素数组
      let length = rewriters_array.length();
      println!("rewriters_array: {}", length);
      for i in 0..length {
        // TODO check unwrap
        let rewriter_pair_value = rewriters_obj.get_index(scope, i as u32).unwrap();
        let rewriter_pair = rewriter_pair_value.to_object(scope).unwrap();

        // 提取标签和处理函数
        let tag_name_js = rewriter_pair
          .get_index(scope, 0)
          .unwrap()
          .to_rust_string_lossy(scope);
        println!("rewriters_array: name {}", tag_name_js);

        let handler_js = rewriter_pair.get_index(scope, 1).unwrap();
        let handle_cb_clone = RefCell::clone(&handle_cb);
        let scope = unsafe { &mut *scope_ptr };
        let element_closure_global = move |el: &mut Element| -> HandlerResult {
          let handle_cb = RefCell::clone(&handle_cb_clone);
          let mut js_args = vec![];
          let js_proxy_obj = element::create_js_element_proxy(scope, el);
          let tag_name = v8::String::new(scope, el.tag_name().as_str()).unwrap();
          js_args.push(tag_name.to_v8());
          js_args.push(js_proxy_obj.to_v8());

          let receiver = js_proxy_obj;
          let result = handle_cb
            .borrow()
            .open(scope)
            .call(scope, receiver.into(), &js_args);
          println!("element_closure is called2");
          Ok(())
        };

        // 存储每个标签和它的处理函数
        element_content_handlers.push(element!(tag_name_js, element_closure_global));
      }
    }
  }

  // for rt in rewriter_type.iter() {
  //   let name = v8::String::new(scope, rt).unwrap();
  //   let rewriters = object.get(scope, name.into());

  //   if let Some(rewriter_array) = rewriters {}
  // }

  // let element_key = v8::String::new(scope, "element").unwrap();
  // let element_obj = object
  //   .get(scope, element_key.into())
  //   .unwrap()
  //   .to_object(scope)
  //   .unwrap();
  // let element_handler = element_obj
  //   .get_own_property_names(scope, Default::default())
  //   .unwrap();

  // let mut element_content_handlers = vec![];

  // let scope_ptr = scope as *mut v8::HandleScope;
  // let handle_cb = RefCell::new(handle_cb);

  // for i in 0..element_handler.length() {
  //   let scope = unsafe { &mut *scope_ptr };
  //   let key = element_handler
  //     .get_index(scope, i)
  //     .unwrap()
  //     .to_rust_string_lossy(scope);
  //   let handle_cb_clone = RefCell::clone(&handle_cb);
  //   let element_closure_global = move |el: &mut Element| -> HandlerResult {
  //     let handle_cb = RefCell::clone(&handle_cb_clone);
  //     let mut js_args = vec![];
  //     let js_proxy_obj = element::create_js_element_proxy(scope, el);
  //     let tag_name = v8::String::new(scope, el.tag_name().as_str()).unwrap();
  //     js_args.push(tag_name.to_v8());
  //     js_args.push(js_proxy_obj.to_v8());

  //     let receiver = js_proxy_obj;
  //     let result = handle_cb
  //       .borrow()
  //       .open(scope)
  //       .call(scope, receiver.into(), &js_args);
  //     println!("element_closure is called2");
  //     Ok(())
  //   };

  //   element_content_handlers.push(element!(key, element_closure_global));
  // }

  let mut output = vec![];

  let settings = Settings {
    element_content_handlers: element_content_handlers,
    ..Settings::default()
  };

  let rewriter_sink: Box<dyn FnMut(&[u8])> = Box::new(move |c: &[u8]| {
    let scope = unsafe { &mut *scope_ptr };
    println!("after process c: {:?}", String::from_utf8(c.to_vec()));

    // let js_c = v8::Uint8Array::new_with_backing_store(scope, &v8::BackingStore::new_copy(c)).into();

    // 首先创建一个新的复制字节slice的 ArrayBuffer (BackingStore)
    let backing_store =
      v8::ArrayBuffer::new_backing_store_from_boxed_slice(c.to_vec().into_boxed_slice());
    let backing_store_shared = backing_store.make_shared();
    let array_buffer = v8::ArrayBuffer::with_backing_store(scope, &backing_store_shared);

    // 根据 ArrayBuffer 创建 Uint8Array
    let uint8_array = v8::Uint8Array::new(scope, array_buffer, 0, c.len()).unwrap();
    let uint8_value = uint8_array.into();

    let js_args = vec![uint8_value];
    let js_args_slice = js_args.as_slice();

    receiver
      .open(scope)
      .call(scope, global.into(), &js_args_slice);
    output.extend_from_slice(c);
    println!(
      "after process output: {:?}",
      String::from_utf8(output.to_vec())
    );
  });

  Ok(HtmlRewriter::new(settings, rewriter_sink))
}
```





doc_end:

```rust

create_js_function!(
  append2,
  DOCEND_DATA,
  DocumentEnd,
  |de: &mut DocumentEnd, scope: &mut v8::HandleScope, args: v8::FunctionCallbackArguments| {
    println!("doc_end.rs: append, {:p}, {:?}", scope, scope);
    let (c, t) = extract_content_and_type(scope, args);
    de.append(&c, t)
  }
);
```





utils.rs

```rust

pub fn make_doctype_closure2<'a>(
  scope_ptr: *mut v8::HandleScope<'static>,
  handle: RefCell<v8::Global<v8::Function>>,
  selector_name: String,
  tag_name: String,
  // func: fn(&mut v8::HandleScope<'static>, &mut Doctype) -> v8::Local<'a, v8::Object>,
) -> impl FnMut(&mut Doctype) -> HandlerResult + 'a {
  move |t: &mut Doctype| -> HandlerResult {
    let scope = unsafe { &mut *scope_ptr };
    let tc_scope = &mut v8::TryCatch::new(scope);

    let handle_cb = RefCell::clone(&handle);
    let js_proxy_obj = doc_type::create_js_proxy(tc_scope, t);
    let selector_name = v8::String::new(tc_scope, selector_name.as_str()).unwrap();
    let tag_name = v8::String::new(tc_scope, tag_name.as_str()).unwrap();
    let mut js_args = vec![];
    js_args.push(selector_name.into());
    js_args.push(tag_name.into());
    js_args.push(js_proxy_obj.into());
    let receiver = js_proxy_obj;
    handle_cb
      .borrow()
      .open(tc_scope)
      .call(tc_scope, receiver.into(), &js_args);

    if tc_scope.exception().is_some() {
      let msg = tc_scope.exception().unwrap().to_rust_string_lossy(tc_scope);
      return Err(msg.into());
    }

    Ok(())
  }
}

pub fn make_docend_closure2<'a>(
  scope_ptr: *mut v8::HandleScope<'static>,
  handle: RefCell<v8::Global<v8::Function>>,
  selector_name: String,
  tag_name: String,
  // func: fn(&mut v8::HandleScope<'static>, &mut Doctype) -> v8::Local<'a, v8::Object>,
  receive_cb: RefCell<v8::Global<v8::Function>>,
) -> impl FnMut(&mut DocumentEnd) -> HandlerResult + 'a {
  move |de: &mut DocumentEnd| -> HandlerResult {
    let tc_scope = unsafe { &mut *scope_ptr };
    // let tc_scope = &mut v8::TryCatch::new(scope);
    let receive_cb = RefCell::clone(&receive_cb);
    let handle_cb = RefCell::clone(&handle);
    let js_proxy_obj = doc_end::create_js_proxy(tc_scope, de, receive_cb);
    let selector_name = v8::String::new(tc_scope, selector_name.as_str()).unwrap();
    let tag_name = v8::String::new(tc_scope, tag_name.as_str()).unwrap();
    let mut js_args = vec![];
    js_args.push(selector_name.into());
    js_args.push(tag_name.into());
    js_args.push(js_proxy_obj.into());
    let receiver = js_proxy_obj;

    handle_cb
      .borrow()
      .open(tc_scope)
      .call(tc_scope, receiver.into(), &js_args);

    // if tc_scope.exception().is_some() {
    //   let msg = tc_scope.exception().unwrap().to_rust_string_lossy(tc_scope);
    //   return Err(msg.into());
    // }

    Ok(())
  }
}







pub fn extract_content_and_type(
  scope: &mut v8::HandleScope,
  args: v8::FunctionCallbackArguments,
) -> (String, ContentType) {
  let content = args.get(0).to_rust_string_lossy(scope);
  let ct = args.get(1).to_object(scope);
  println!("ct: {:?}", ct);
  println!("ct: {:?}", ct.is_none());
  if ct.is_none() {
    return (content, ContentType::Html);
  }
  let key = v8::String::new(scope, "html").unwrap();
  let html = ct.unwrap().get(scope, key.into());
  if html.is_none() || !html.unwrap().is_boolean() {
    let message = v8::String::new(scope, "extract_content_and_type error2").unwrap();
    let exception = v8::Exception::type_error(scope, message);
    scope.throw_exception(exception);
    // return (content, ContentType::Html);
  }

  let html_bool = html.unwrap().to_boolean(scope);
  if html_bool.is_true() {
    return (content, ContentType::Html);
  } else {
    return (content, ContentType::Text);
  }
}
```


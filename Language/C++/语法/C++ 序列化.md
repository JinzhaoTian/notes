
### Google Protocol Buffers


### Boost.Serialization


### MFC Serialization


### MessagePack

[MessagePack](https://msgpack.org/) 同样也支持C++

```c++
struct Person {
  std::string name;
  uint16_t age;
  std::vector<std::string> aliases;

  template<class T>
  void msgpack(T &pack) {
    pack(name, age, aliases);
  }
};

int main() {
    auto person = Person{"John", 22, {"Ripper", "Silverhand"}};

    auto data = msgpack::pack(person); // Pack your object
    auto john = msgpack::unpack<Person>(data.data()); // Unpack it
}
```
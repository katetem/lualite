## Description ##
A `C++` library for generating `Lua` bindings.

Supported features:
  * functions,
  * constructors,
  * inheritance,
  * member functions,
  * properties,
  * standard containers,
  * user types.

## Example ##
```
  lualite::module{L,
    lualite::class_<testbase>("testbase")
      .constant("__classname", "testbase")
      .constant("__b", true)
      .constant("__pi", 3.1459)
      .def<decltype(&testbase::dummy), &testbase::dummy>("dummy"),
    lualite::class_<testclass>("testclass")
      .constructor("defaultNew")
      .constructor<int>()
      .inherits<testbase>() // you can add more classes to inherit from
      .enum_("smell", 9)
      .def<std::tuple<int, std::string, char const*> (testclass::*)(int), &testclass::print>("print")
      .def<std::vector<std::string> (testclass::*)(std::string) const, &testclass::print>("print_")
      .def<LLFUNC(testclass::pointer)>("pointer")
      .def<LLFUNC(testclass::reference)>("reference")
      .property<LLFUNC(testclass::a), LLFUNC(testclass::set_a)>("a")
      .def<LLFUNC(testclass::test_array)>("test_array"),
    lualite::scope("subscope",
      lualite::class_<testclass>("testclass")
        .constructor<>("defaultNew")
        .constructor<int>()
        .enum_("smell", 10)
        .def<LLFUNC(testfunc)>("testfunc")
        .def<std::tuple<int, std::string, char const*> (testclass::*)(int), &testclass::print>("print")
        .def<std::vector<std::string> (testclass::*)(std::string) const, &testclass::print>("print_")
    )
  }
  .enum_("apple", 1)
  .def<LLFUNC(testfunc)>("testfunc")
  .def<LLFUNC(testpair)>("testpair")
  .def<LLFUNC(testtuple)>("testtuple");

  luaL_dostring(
    L,
    "local a = testfunc(3, 2, 1)\n"
    "r = { \"my\", \"pair\" }\n"
    "testpair(r)\n"
    "r[3] = 3\n"
    "testtuple(r)\n"
    "print(a.y)\n"
    "print(apple)\n"
    "print(testclass.smell)\n"
    "print(testbase.__classname)\n"
    "print(testbase.__b)\n"
    "print(testbase.__pi)\n"
    "local b = testclass.defaultNew()\n"
    "print(\"---\")\n"
    "print(b.a)\n"
    "b:reference().a = 888\n"
    "print(b.dummy)\n"
    "print(b.a .. \" \" .. b:dummy(\"test\"))\n"
    "local tmp1, tmp2, tmp3 = b:pointer():print(100)\n"
    "print(tmp1 .. \" \" .. tmp2 .. \" \" .. tmp3)\n"
    "b:reference():print_(\"msg1\")\n"
    "local a = subscope.testclass.new(1111)\n"
    "print(subscope.testclass.smell)\n"
    "subscope.testclass.testfunc(200, 0, 1)\n"
    "local c = a:reference():print_(\"msg2\")\n"
    "print(c[10])\n"
    "r = {}"
    "for i = 1, 10 do\n"
    "  r[i] = 7\n"
    "end\n"
    "print(a:test_array(r))\n"
  );
```

Try it [online](http://lualitedemo.square7.ch/).

## FAQ ##

**Q:** Why can't I return `char*`:

try returning `char const*`. Don't return references or pointers to non-const objects, if you don't provide wrappers for them (returning a non-const reference or pointer implies ability to change the referred object inside `lua`, which is not possible without also writing a wrapper class for the type of the referred-to object).

**Q:** Why am I getting:

`Assertion failed: (sizeof...(A) + O - 1 == lua_gettop(L)), function member_stub, file ../src/Script/LuaLite.h, line 1172.`

whenever I try to call a member function declared here:

```
class TestClass {
public:
	TestClass();
	virtual ~TestClass();

	void sayHello();

	static void registerFunc(lua_State* L) {
		lualite::module(L,
			lualite::class_<TestClass>("TestClass")
			.constructor("new")
			.def<LLFUNC(TestClass::sayHello)>("sayHello"));
	}
};
```

using this `lua` code:

```
luaL_dostring(luaState,
	"local test = TestClass.new()\n"
	"test.sayHello()");
```

**A:** Try `test:sayHello()`. Or try:
```
			.def_func<LLFUNC(TestClass::sayHello)>("sayHello"));
```
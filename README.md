# Piqla v1.1.19

## Available methods

```php
function Pinq(array $input = []): Pinq; // Constructor

function where(callable ...$funcs): Pinq;
function select(callable ...$funcs): Pinq;
function delete(callable ...$funcs): Pinq;
function update(callable ...$funcs): Pinq;
function insert(callable ...$funcs): Pinq;
function orderBy(boolean $accending, callable ...$funcs): Pinq;
function orderAscendingBy(callable ...$funcs): Pinq;
function orderDescendingBy(callable ...$funcs): Pinq;
function distinct(): Pinq;
function join(array $list, callable $select, callable ...$wheres): Piqla;
function group(callable $heads, callable ...$funcs): Piqla[];
function count(): int;
function min(callable ...$funcs): Pinq;
function max(callable ...$funcs): Pinq;
function sum(callable ...$funcs): Pinq;
function average(callable ...$funcs): Pinq;
function limit(int $count, int $offset): Pinq;
function offset(int $offset): Pinq;

function fromJsonString(string $json);
function toJsonString(): string;
function toArray(): array;
```

## How to use

Define an array and pass that to new Piqla instance constructor

Callback functions must have one argument at least
This argument (its name doen't matter) is current item in list in loop
After that, other else arguments must match with list members name to pass value of that member instead of that argument and these arguments are not required and you have not to define argument for each member beacause you can access by first argument to all of members and other arguments use for easy access to some members.

Now you can use Piqla deformer functions and call them on returned result again and again and ... (like bellow samples)

## Sample

```php
$persons = new Piqla([
    [
        "_id" => 1,
        "name" => "jack",
        "family" => "gonjishke",
        "age" => 45
    ],
    [
        "_id" => 2,
        "name" => "joe",
        "family" => "gandomi",
        "age" => 32
    ],
    [
        "_id" => 3,
        "name" => "jack",
        "family" => "landan",
        "age" => 23
    ],
    [
        "_id" => 4,
        "name" => "john",
        "family" => "val john",
        "age" => 63
    ]
]);

$addresses = new Piqla([
    ["_id" => 1, "person_id" => 2, "phone_number" => "12398747", "country" => "near", "city" => "here", "street" => "here 1", "since" => "2017/02/12"],
    ["_id" => 2, "person_id" => 1, "phone_number" => "22118965", "country" => "far", "city" => "there", "street" => "there 12", "since" => "2015/07/03"],
    ["_id" => 3, "person_id" => 2, "phone_number" => "55663322", "country" => "near", "city" => "here", "street" => "here 2", "since" => "2018/03/01"],
    ["_id" => 4, "person_id" => 3, "phone_number" => "74653689", "country" => "far", "city" => "right of there", "street" => "rot 3", "since" => "2016/05/23"],
    ["_id" => 5, "person_id" => 1, "phone_number" => "77441122", "country" => "near", "city" => "near of hear", "street" => "noh 23", "since" => "2017/10/11"]
]);
```

### Test functions

#### where()

```php
echo "<h3>Where:</h3>";
$persons->where(function ($item, $name, $family, $age) {
    return ($age > 35);
});

// Result: [{name: "jack", family: "gonjishke", age: 45},{name: "john", family: "val john", age: 63}]
```

#### select()

```php
$persons->select(function ($item, $name, $family, $age) {
    return ["fullname" => $family . ", " . $name, "old" => ($age > 40)];
});

// Result: [fullname: "gonjishke, jack", old: true}, {fullname: "gandomi, joe", old => false}, {fullname: "landan, jack" old: false}, {fullname: "val john, john", old: true}]


$persons->select(function ($item, $name, $family, $age) {
    if ($age > 40) return ["fullname" => $family . ", " . $name];
});

// Result: [fullname: "gonjishke, jack", {fullname: "val john, john"}]
```

#### delete()

```php
$persons->delete(function ($item, $name, $family, $age) {
    return $age < 40;
});

// Result: [name: "jack", family: "gonjishke", age: 45}, {name: "john", family: "val john", age: 63}]
```

#### update()

```php
$persons->update(function ($item, $name, $family, $age) {
    if ($age > 40)
        return ["age" => round($age / 2), "old" => true];
    else
        return ["old" => false];
});

// Result: [name: "jack", family: "gonjishke", age: 23, old: true}, {name: "joe", family: "gandomi", age: 32, old: false}, {name: "jack", family: "landan", age: 23, old: false}, {name: "john", family: "val john", age: 32, old: true}]
```

#### insert()

```php
$persons->insert(function () {
    return ["name" => "nicol", "family" => "cadmiom", "old" => true];
});

// Result: [name: "jack", family: "gonjishke", age: 45, old: NULL}, {name: "joe", family: "gandomi", age: 32, old: NULL}, {name: "jack", family: "landan", age: 23, old: NULL}, {name: "john", family: "val john", age: 63, old: NULL}, {name: "nicol", family: "cadmiom", old: true, age: NULL}]
```

#### orderBy()

```php
$persons->orderDescendingBy(function ($item, $name, $family, $age) {
    return [$age > 35, -$age];
});

// Result: [name: "jack", family: "gonjishke", age: 45}, {name: "john", family: "val john", age: 63}, {name: "jack", family: "landan", age: 23}, {name: "joe", family: "gandomi", age: 32}]
```

#### distinct()

```php
$persons->name->distinct();

// Result: [name: "jack"}, {name: "joe"}, {name: "john"}]
```

#### join()

```php
$persons->join($addresses->toArray(), function ($left, $right) {
    return ["full_name" => $left["family"] . ", " . $left["name"], "phone_number" => $right["phone_number"], "address" => $right["street"] . ", " . $right["city"] . ", " . $right["country"]];
}, function ($left, $right) {
    return ($left["_id"] == $right["person_id"]);
});

// Result: [{"full_name":"gonjishke, jack","phone_number":"22118965","address":"there 12, there, far"},{"full_name":"gonjishke, jack","phone_number":"77441122","address":"noh 23, near of hear, near"},{"full_name":"gandomi, joe","phone_number":"12398747","address":"here 1, here, near"},{"full_name":"gandomi, joe","phone_number":"55663322","address":"here 2, here, near"},{"full_name":"landan, jack","phone_number":"74653689","address":"rot 3, right of there, far"}]
```

#### group()

```php
$addresses->group(function ($item) {
    return $item["country"];
}, function ($item) {
    return ["_id" => $item["_id"], "person_id" => $item["person_id"]];
});

// Result: {"near":[{"_id":1,"person_id":2},{"_id":3,"person_id":2},{"_id":5,"person_id":1}],"far":[{"_id":2,"person_id":1},{"_id":4,"person_id":3}]}
```

#### count()

```php
$persons->count();

// Result: 4
```

#### min()

```php
$persons->min(function ($item, $age) {
    if ($item["name"] == "jack") {
        return $age;
    }
}, function ($item) {
    if ($item["name"] != "jack") {
        return strlen($item["name"]);
    }
});

// Result: [name: "jack", family: "landan", age: 23}, {name: "joe", family: "gandomi", age: 32}]
```

#### max()

```php
$persons->max(function ($item, $age) {
    if ($item["name"] == "jack") {
        return $age;
    }
}, function ($item) {
    if ($item["name"] != "jack") {
        return strlen($item["name"]);
    }
});

// Result: [name: "jack", family: "gonjishke", age: 45}, {name: "john", family: "val john", age: 63}]
```

#### sum()

```php
$persons->sum(function ($item, $age) {
    if ($item["name"] == "jack") {
        return $age;
    }
}, function ($item) {
    if ($item["name"] != "jack") {
        return strlen($item["name"]);
    }
})->sum();

// Result: [75]
```

#### average()

```php
$persons->average(function ($item, $age) {
    if ($item["name"] == "jack") {
        return $age;
    }
}, function ($item) {
    if ($item["name"] != "jack") {
        return strlen($item["name"]);
    }
});

// Result: [34, 3.5]
```

#### limit()

```php
$persons->limit(2, 1);

// Result: [name: "joe", family: "gandomi", age: 32}, {name: "jack", family: "landan", age: 23}]
```

#### offset()

```php
$persons->offset(2);

// Result: [name: "jack", family: "landan", age: 23}, {name: "john", family: "val john", age: 63}]
```

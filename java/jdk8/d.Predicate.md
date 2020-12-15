接口定义：

```java
@FunctionalInterface
public interface Predicate<T> {

    /**
     * 输入一个参数，返回一个boolean值
     */
    boolean test(T t);

    /**
     * 输入一个Predicate对象，与源对象组成and关系
     */
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    /**
     * 逻辑非关系
     */
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    /**
     * 输入一个Predicate对象，与源对象组成or关系
     */
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    /**
     * 判断是否相等，依据Objects.equals
     */
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}

```

示例程序：

```java
public class PredicateTest {
    public static void main(String[] args) {
        //使用Lambda表达式定义Predicate
        Predicate<String> predicate = p -> p.length() > 5;
        System.out.println(predicate.test("hello"));

        //传入Predicate实现
        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
        PredicateTest predicateTest = new PredicateTest();
        predicateTest.conditionFilter(list,item -> item % 2 == 0);
        System.out.println("--------------------------------");
        predicateTest.conditionFilter(list,item -> item > 5);
        //Predicate.and(),使用and组合两个表达式
        System.out.println("--------------------------------");
        predicateTest.conditionFilter(list,item -> item > 5 ,item -> item % 2 == 0);
    }

    public void conditionFilter(List<Integer> list,Predicate<Integer> predicate){
        for (Integer integer : list){
            if(predicate.test(integer)){
                System.out.println(integer);
            }
        }
    }

    public void conditionFilter(List<Integer> list,Predicate<Integer> predicate1,Predicate<Integer> predicate2){
        for (Integer integer : list){
            if(predicate1.and(predicate2).test(integer)){
                System.out.println(integer);
            }
        }
    }
}
```

与stream流的filter方法一起用：

```java
public class PersonTest {
    public static void main(String[] args) {
        Person person = new Person("zhangsan",20);
        Person person1 = new Person("lisi",30);
        Person person2 = new Person("wangwu",40);
        List<Person> persons = Arrays.asList(person,person1,person2);
        PersonTest test = new PersonTest();
        //使用流过滤出姓名等于zhangsan的person
        List<Person> personResult = test.getPersonByUserName("zhangsan",persons);
        System.out.println(personResult);
        //使用流过滤出年龄大于20的person，传入行为
        List<Person> personResult1 = test.getPersonByAge(20,persons,(age,personList) ->
                personList.stream().filter( thePerson -> thePerson.getAge() > age).collect(Collectors.toList()) );
        System.out.println(personResult1);

    }

    public List<Person> getPersonByUserName(String userName,List<Person> list){
        return list.stream().filter(person -> person.getUserName().equals(userName)).collect(Collectors.toList());
    }

    public List<Person> getPersonByAge(int age,List<Person> persons, BiFunction<Integer,List<Person>,List<Person>> biFunction){
        return biFunction.apply(age,persons);
    }
}


class Person{
    private String userName;
    private Integer age;

    public Person(String userName, Integer age) {
        this.userName = userName;
        this.age = age;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "userName='" + userName + '\'' +
                ", age=" + age +
                '}';
    }
}
```


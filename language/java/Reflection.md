# Reflection

### **JVM에서 실행되는 어플리케이션을 동적으로 검사하거나 수정할 수 있는 기능**

리플랙션은 주로 Class<T>의 접근으로 이루어진다.

Class<T>는 ClassLoader가 클래스를 로딩한 후 Class<T> 클래스를 heap 영역에 올린다.

### Class<T>에 접근하는 방법

1. 타입.class로 접근

    ```java
    Class<Point> cls = Point.class; 
    ```

2. instance의 getClass() 메소드 이용

    ```java
    public class Point {
        private final int x;
        private final int y;
    
        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    
        public static void main(String[] args) {
            Point p = new Point(1, 2);
            Class<? extends Point> cls = p.getClass();
        }
    }
    ```

   instance의 getClass 메소드의 리턴 값은 ? extends [type]

3. Class.forName([FQCN])

   FQCN —> Class의 패키지를 포함한 모든 경로를 명시

   리턴 타입은 Class<?> 명시적인 형변환 필요

    ```java
    Class<?> cls = Class.forName("Point");
    ```


### Reflection으로 할 수 있는 기능

**private 경우 setAccessible(true)로 선언해주어야한다. (Method 필드 다)**

1. 필드(목록) 가져오기

    ```java
    public class Point {
        private final int x;
        private final int y;
    
        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    
        public static void main(String[] args){
            Class<Point> cls = Point.class;
            /**
             * getFields는 제어자에 따라 접근 가능한 정보만 볼 수 있다.
             * 결과 출력 x
             */
            Arrays.stream(cls.getFields()).forEach(System.out::println);
    
            /**
             * getDeclaredFields는 선언되어있는 모든 필드에접근 (접근 제어자 무시)
             * private final int Point.x
             * private final int Point.y
             */
            Arrays.stream(cls.getDeclaredFields()).forEach(System.out::println);
        }
    }
    ```

2. 메소드(목록) 가져오기

    ```java
    public class Point {
        private final int x;
        private final int y;
    
        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    
        private void privateMethod() {
            System.out.println("Private");
        }
    
        public int getX() {
            return x;
        }
    
        public int getY() {
            return y;
        }
    
        public static void main(String[] args) throws ClassNotFoundException {
            Class<Point> cls = Point.class;
            /**
             * getMethods는 제어자에 따라 접근 가능한 정보만 볼 수 있다.
             * 상속받은 Method까지 가져온다.
             */
            Arrays.stream(cls.getMethods()).forEach(System.out::println);
    
            /**
             * getDeclaredMethods는 선언되어있는 모든 필드에접근 (접근 제어자 무시)\
             * 접근제어자를 무시하고 선언된 모든 Method를 가져온다.
             * 하지만 상속받은 Method는 무시한다.
             */
            System.out.println("Declared");
            Arrays.stream(cls.getDeclaredMethods()).forEach(System.out::println);
        }
    ```

3. 상위 클래스 가져오기

    ```java
    public class Point {
        private final int x;
        private final int y;
    
        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
        public static void main(String[] args){
            Class<ColoredPoint> cls = ColoredPoint.class;
            Class c = cls;
            do {
                c = c.getSuperclass();
                System.out.println(c);
            } while (c != null);
        }
        class ColoredPoint extends Point {
            public ColoredPoint(int x, int y) {
                super(x, y);
            }
        }
    }
    ```

4. 인터페이스(목록) 가져오기

    ```java
    public class Point {
        private final int x;
        private final int y;
    
        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    
        private Point() {
            x = 10;
            y = 10;
        }
    
        public static void main(String[] args) {
            Class<Point> cls = Point.class;
            Arrays.stream(cls.getInterfaces()).forEach(System.out::println);
        }
    }
    ```

5. 에노테이션 가져오기

    ```java
    public class Point {
        private final int x;
        private final int y;
    
        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    
        private Point() {
            x = 10;
            y = 10;
        }
    
        public static void main(String[] args) {
            Class<Point> cls = Point.class;
            Arrays.stream(cls.getAnnotations()).forEach(System.out::println);
        }
    }
    ```

6. 생성자 가져오기

    ```java
    public class Point {
        private final int x;
        private final int y;
    
        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    
        private Point() {
            x = 10;
            y = 10;
        }
    
        public static void main(String[] args){
            Class<Point> cls = Point.class;
            /**
             * public Constructor만 출력
             */
            Arrays.stream(cls.getConstructors()).forEach(System.out::println);
            /**
             * private Constructor만 출력
             */
            Arrays.stream(cls.getDeclaredConstructors()).forEach(System.out::println);
        }
    }
    ```

7. 메소드 실행

    ```java
    public class Point {
        private final int x;
        private final int y;
    
        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    
        private Point() {
            x = 10;
            y = 10;
        }
    
        public void print() {
            System.out.println(this.x + " " + this.y);
        }
    
        public static void main(String[] args) throws InstantiationException, NoSuchMethodException, InvocationTargetException, IllegalAccessException {
            Class<Point> cls = Point.class;
            Point p = cls.getConstructor(int.class, int.class).newInstance(20, 30);
            cls.getMethod("print").invoke(p);
        }
    /*
    	invoke(Object obj, Object... args)
    	첫번째 인자는 instance (static Method라면 없어도된다)
    	인자가 없다면 생략가능
    */
    }
    ```

8. 인스턴스 생성

    ```java
    public class Point {
        private final int x;
        private final int y;
    
        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    
        private Point() {
            x = 10;
            y = 10;
        }
    
        public void print() {
            System.out.println(this.x + " " + this.y);
        }
    
        public static void main(String[] args) throws InstantiationException, NoSuchMethodException, InvocationTargetException, IllegalAccessException {
            Class<Point> cls = Point.class;
            Point p = cls.getConstructor(int.class, int.class).newInstance(20, 30);
        }
    /*
    	Class.newInstance()는 deprecated되었기 때문에, constructor를 통해서 실행해야한다.
    */
    }
    ```


이 외에도 Reflection에는 많은 기능이 있다.

IDE의 자동완성 스프링,JPA 등 많은 곳에서 사용된다.
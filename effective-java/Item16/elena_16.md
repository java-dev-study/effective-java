## **public 클래스에서는 접근자 메서드 사용**

```java
class Point {
  public double x;
  public double y;
}
```
- 데이터 필드에 직접 접근 가능
- API를 변경하지 않고는 내부 표현 수정 불가
- 불변식 보장 불가
- 외부에서 필드 접근 시 부수 작업 수행 불가\

↓\
`필드 private & getter 추가`
```java
class Point {
  private double x;
  private double y;
  
  public Point(double x, double y) {
    this.x = x;
    this.y = y;
  }
  
  public double getX() { return x; }
  public double getY() { return y; }
  
  public void setX(double X) { this.x = x; }
  public void setY(double Y) { this.y = y; }
}
```
- 접근자 제공 → 유연성
---
- public class의 필드가 불변일 경우
> 1. API를 변경하지 않고는 표현 방식 수정 불가
> 2. 필드를 읽을 때 부수 작업 수행 불가
> 3. 불변식 보장

↓\
`각 인스턴스가 유효한 시간을 표현함을 보장하는 코드`
```java
public final class Time {
  private static final int HOURS_PER_DAY    = 24;
  private static final int MINUTES_PER_HOUR = 60;
  
  public final int hour;
  public final int minute;
  
  public Time(int hour, int minute) {
    if(hour < 0 || hour >= HOURS_PER_DAY) {
      thorw new IllegalArgumentException("시간: " + hour);
    }
    if(minute < 0 || minute >= MINUTES_PER_HOUR) {
      throw new IllegalArgumentException("분: " + minute);
    }
    this.hour = hour;
    this.minute = minute;
  }
  ...
}
```

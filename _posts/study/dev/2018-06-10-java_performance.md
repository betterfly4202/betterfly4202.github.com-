---
title: "반복문 성능 이야기"
categories: "JAVA"
tags:
---

뒤늦게 Java Stream에 관심이 생기면서, forEach의 성능에 대한 이슈를 접하게 되었다.
[for-loop를 Stream.forEach()로 바꾸지 말아야 할 3가지 이유](https://homoefficio.github.io/2016/06/26/for-loop-%EB%A5%BC-Stream-forEach-%EB%A1%9C-%EB%B0%94%EA%BE%B8%EC%A7%80-%EB%A7%90%EC%95%84%EC%95%BC-%ED%95%A0-3%EA%B0%80%EC%A7%80-%EC%9D%B4%EC%9C%A0/)

그래서 비교해보았다. 일반적으로 사용하는 반복문
- While
- Iterator
- for
- for-loop
- for each

총 5개의 반복문에 대한 이야기이다.

테스트 케이스는 iterator 및 for-each를 위해서 List<Integer>로 정의했으며, 100개의 난수 리스트를 생성하여, 리스트 각 인덱스의 총 합을 반복문으로 구현했다.

우선 시간을 계산해야하니까 결과를 ms으로 반환해주는 시간 측정 메서드이다.

~~~java
class TimerUtil {
    long time;

    public void start(){
        time = System.nanoTime();
    }

    public double end(){
//        System.out.println("compute result : " + (System.nanoTime()-time)/ 1000000.0);
        return (System.nanoTime()-time)/ 1000000.0;
    }
}
~~~

---

다음은 난수를 저장하는 리스트이다.

~~~java
    private static List<Integer> intStream(){
        List<Integer> arrList = new ArrayList<>();
        IntStream.range(0, 100)
                .forEach(i -> arrList.add(generator(100)));

        return arrList;
    }

    private static int generator(int max){
        Random random = new Random();
        return random.nextInt(max);
    }
~~~

그리고 각 반복문 내용이다.
~~~java
    private static double performanceByWhile(List<Integer> array){
        /*
            while
         */
        result = 0;

        timer.start();
        int s=0;
        while(s<array.size()){
            result += array.get(s);
            s++;
        }
        return timer.end();
    }

    private static double performanceByFor(List<Integer> array){
        /*
            for
         */
        result = 0;

        timer.start();
        for(int i=0; i<array.size(); i++){
            result += array.get(i);
        }
        return timer.end();
    }

    private static double performanceByForLoop(List<Integer> array){
        /*
            for loop
         */
        result = 0;

        timer.start();
        for(int temp : array){
            result += temp;
        }
        return timer.end();
    }

    private static double performanceByForEach(List<Integer> array){
        /*
            for each
         */
        result = 0;

        timer.start();
        array.forEach((temp) -> {
            result += temp;
        });

        return timer.end();
    }


    private static double performanceByIterator(List<Integer> array){
        /*
            iterator
         */
        result = 0;

        timer.start();

        Iterator<Integer> itr = array.iterator();
        while( itr.hasNext() ){
            result += array.get(itr.next());
        }

        return timer.end();compare_Repetition.jpeg
    }

~~~


최종적으로 이 모든 과정을 테스트하는 메인 메서드.

~~~java
    public static void main (String [] args){
        List<Integer> sumArr = intStream();

        int maxCnt = 10000;
        double sumTimes = 0;
        for(int cnt = 0; cnt<maxCnt; cnt++){
        sumTimes += performanceByFor(sumArr);
        // sumTimes += performanceByWhile(sumArr);
        // sumTimes += performanceByIterator(sumArr);
        // sumTimes += performanceByForLoop(sumArr);
        // sumTimes += performanceByForEach(sumArr);
        
        }
        sumTimes *= 1000;
        System.out.println("Result Time : "+resultAvg(sumTimes, maxCnt));

    }
~~~

각 반복문의 평균 수행 시간(ms) 결과이다.
![result](/assets/images/180615/compare_Repetition.jpeg)


---
> 테스트 결과로는 **While=for > For-loop > For-each > Iterator** 순으로 보인다.

올바른 방법으로 테스트한것인지는 검증이 필요하겠지만, 서두에 링크로 참고해두었던 본문 내용에서도 말했듯이 for-each와 같이 Stream을 사용할 경우 누적되는 오버헤드 비용이 많이 발생하기 때문에 큰 사이즈를 반복문 처리할때는 유의해야할 필요가 있을 것으로 보인다.

각각의 역할이 필요한 곳에 따라 유연하게 사용할 수 있도록 보다 확실하게 개념을 공부해야 할 것 같다.

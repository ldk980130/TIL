## [Configuring a Job](https://docs.spring.io/spring-batch/docs/current/reference/html/job.html#configuringAJob)

`Job` 인터페이스에는 여러 구현체들이 존재한다. 하지만 builder는 구성의 차이를 추상화한다.

```java
@Bean
public Job footballJob() {
    return this.jobBuilderFactory.get("footballJob")
                     .start(playerLoad())
                     .next(gameLoad())
                     .next(playerSummarization())
                     .build();
}
```

`Job`(보통 `Step`도 함께 정의됨)은 `JobRepository`를 필요로 한다. `BatchConfigure`를 통해 `JobRepository` configuration이 처리된다.

위 코드는 세 개의 `Step` 인스턴스로 구성된 `Job`을 보여준다. `Job`과 관련된 빌더는 병렬화(`Split`), 선언적 흐름 제어(`Decision`), 흐름 정의의 외부화(`Flow`) 등을 돕는 다른 요소도 포함할 수 있다.

### [Restartability](https://docs.spring.io/spring-batch/docs/current/reference/html/job.html#restartability)

배치 작업을 실행할 때 발생하는 주요 문제 중 하나는 작업이 재시작될 때의 동작과 관련 있다. 특정 `JobInstance`에 대한 `JobExecution`이 이미 존재하는 경우 `Job`은 ‘재시작’으로 간주된다. 이상적으로 모든 작업이 중단된 곳에서 시작할 수 있어야 하지만 이것이 불가능한 시나리오가 있다. *이 시나리오에서 새 `JobInstance`가 생성되는지 확인하는 것은 전적으로 개발자에게 달려 있다.* 하지만 스프링 배치는 약간의 도움을 제공한다. `Job`을 재시작 해서는 안 되고 항상 새 `JobInstance`의 일부로 실행해야 하는 경우 재시작 가능 속성을 ‘false’로 설정할 수 있다.

```java
@Bean
public Job footballJob() {
    return this.jobBuilderFactory.get("footballJob")
                     .preventRestart()
                     ...
                     .build();
}
```

### [Intercepting Job Execution](https://docs.spring.io/spring-batch/docs/current/reference/html/job.html#interceptingJobExecution)

`Job`을 실행하는 과정에서 Custom code가 실행될 수 있도록 라이프사이클의 다양한 이벤트에 대한 알림을 받는것이 유용할 수 있다. `SimpleJob`은 적절한 시간에 `JobListener`를 호출하여 이를 가능하게 한다.

```java
public interface JobExecutionListener {

    void beforeJob(JobExecution jobExecution);

    void afterJob(JobExecution jobExecution);

}
```

```java
@Bean
public Job footballJob() {
    return this.jobBuilderFactory.get("footballJob")
                     .listener(sampleListener())
                     ...
                     .build();
}
```

`afterJob()` 메서드는 작업의 실패 성공 여부와 상관 없이 동작하기 때문에 적절한 처리를 해주어야 한다.

```java
public void afterJob(JobExecution jobExecution){
    if (jobExecution.getStatus() == BatchStatus.COMPLETED ) {
        //job success
    }
    else if (jobExecution.getStatus() == BatchStatus.FAILED) {
        //job failure
    }
}
```

### [JobParametersValidator](https://docs.spring.io/spring-batch/docs/current/reference/html/job.html#jobparametersvalidator)

런타임에 실행 매개 변수 validator를 선택적으로 선언할 수 있다. 이는 어떤 작업이 필수 매개 변수로 시작해야만 할 때 유용하다. `DefaultJobParametersValidator` 인터페이스를 구현하여 추가할 수 있다.

```java
@Bean
public Job job1() {
    return this.jobBuilderFactory.get("job1")
                     .validator(parametersValidator())
                     ...
                     .build();
}
```

## [Java Config](https://docs.spring.io/spring-batch/docs/current/reference/html/job.html#javaConfig)

Spring Batch 2.2.0부터는 동일한 Java 구성을 사용하여 배치 작업을 구성할 수 있다. `@EnableBatchProcessing` 어노테이션과 두 개의 빌더라는 두 구성 요소가 있다.

`@EnableBatchProcessing` 어노테이션은 배치 작업을 구성하는데 필요한 기본 구성을 제공한다.

- `JobRepository`: bean name "jobRepository"
- `JobLauncher`: bean name "jobLauncher"
- `JobRegistry`: bean name "jobRegistry"
- `PlatformTransactionManager`: bean name "transactionManager"
- `JobBuilderFactory`: bean name "jobBuilders"
- `StepBuilderFactory`: bean name "stepBuilders"

`BatchConfigure`가 이 구성의 핵심 인터페이스다. 기본 구현은 위에서 언급한 빈을 제공하며 컨텍스트 내에서 빈으로 `Datasource`를 제공해야 한다.  `Datasorce`는 `JobRepository`가 사용한다. `BatchConfigure`를 재정의하여 기능을 확장할 수도 있다.

```java
@Bean
public BatchConfigurer batchConfigurer(DataSource dataSource) {
	return new DefaultBatchConfigurer(dataSource) {
		@Override
		public PlatformTransactionManager getTransactionManager() {
			return new MyTransactionManager();
		}
	};
}
```

기본 configuration을 사용하면 사용자는 제공된 빌더 팩토리를 사용하여 job을 구성할 수 있다.

```java
@Configuration
@EnableBatchProcessing
@Import(DataSourceConfiguration.class)
public class AppConfig {

    @Autowired
    private JobBuilderFactory jobs;

    @Autowired
    private StepBuilderFactory steps;

    @Bean
    public Job job(@Qualifier("step1") Step step1, @Qualifier("step2") Step step2) {
        return jobs.get("myJob").start(step1).next(step2).build();
    }

    @Bean
    protected Step step1(ItemReader<Person> reader,
                         ItemProcessor<Person, Person> processor,
                         ItemWriter<Person> writer) {
        return steps.get("step1")
            .<Person, Person> chunk(10)
            .reader(reader)
            .processor(processor)
            .writer(writer)
            .build();
    }

    @Bean
    protected Step step2(Tasklet tasklet) {
        return steps.get("step2")
            .tasklet(tasklet)
            .build();
    }
}
```

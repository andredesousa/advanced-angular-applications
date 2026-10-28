# Testing Angular Applications

## Overview

As applications grow in size and complexity, manual validation alone is no longer enough to catch regressions and ensure release confidence. Software testing establishes whether the system behavior matches expectations across functional, integration, and user-level scenarios.

Delivering high-quality software applications requires proper testing. Effective testing improves the quality of the support and services provided to customers. In addition, a well-tested product results in lower maintenance costs and delivers more accurate, consistent, and reliable outcomes. The cost of fixing a bug in production is significantly higher than the cost of detecting it through an automated test suite. When defects are identified early in the development process, developers can resolve them more efficiently and with less effort.

Modern web applications also run across diverse browsers, devices, operating systems, and network conditions. That variability makes systematic testing essential for compatibility, accessibility, and consistent user experience between development and production environments. Black Box Testing and White Box Testing are complementary software testing approaches that validate a system's functionality from the user's perspective and verify its internal code logic and structure, respectively.

This chapter covers the core testing layers for Angular applications, practical test strategy tradeoffs, and implementation patterns that help teams build fast feedback loops while maintaining high product quality over time. On the tooling side, it discusses [Vitest](https://vitest.dev/) and [Jest](https://jestjs.io/) for unit and integration tests, [Playwright](https://playwright.dev/) for functional and visual tests, and [Cucumber](https://cucumber.io/) for Behaviour-Driven Development.

## Software Development Process

### Software Development Life Cycle

[Software Development Process](https://en.wikipedia.org/wiki/Software_development_process) is the practice of organizing software work into defined phases to improve planning, delivery, and quality. It is often described as a [Software Development Life Cycle](https://en.wikipedia.org/wiki/Systems_development_life_cycle) (SDLC), which provides a structured path from idea to production operation.

The primary goal of SDLC is to deliver high-quality software that meets user needs within expected time and budget constraints. Typical SDLC activities include:

- **Planning**: Define goals, constraints, scope, and acceptance criteria;
- **Design**: Define system architecture, data flow, interfaces, and technical decisions;
- **Implementation**: Build the solution according to the approved design and requirements;
- **Testing**: Validate behavior, quality, and reliability, and detect defects early;
- **Documentation**: Maintain technical and functional documentation throughout the lifecycle;
- **Deployment**: Release the software to target environments in a controlled way;
- **Maintenance**: Monitor production behavior, fix defects, and implement iterative improvements.

Common software delivery approaches include [Waterfall](https://en.wikipedia.org/wiki/Waterfall_model) and [Agile](https://en.wikipedia.org/wiki/Agile_software_development). Within Agile, [Scrum](https://en.wikipedia.org/wiki/Scrum_(software_development)) and [Kanban](https://en.wikipedia.org/wiki/Kanban_(development)) are among the most widely used frameworks.

### Behavior-Driven Development

[Behavior-Driven Development](https://en.wikipedia.org/wiki/Behavior-driven_development) (BDD) is a methodology for developing software through continuous example-based communication between developers, QAs and POs. It describes a cycle of interactions with well-defined outputs, resulting in the delivery of working, tested software that matters.

The process looks something like this:

1. Product Owner (PO) and business analyst define features in Gherkin;
2. PO and business analyst together with the team enrich features with scenarios;
3. Team agrees on a user story by its feature definition;
4. Feature definition is run against a tool parsing and executing the Gherkin and will fail;
5. The features are built and test steps implemented so that the feature passes.

The goal of BDD is a business-readable and domain-specific language that allows teams to describe a system’s behavior without explaining how that behavior is implemented. Tests are written in the form of plain-text feature descriptions with scenarios typically written before anything else and verified by non-technical stakeholders. No development skills are required to create tests, as they are written in a universal language. Business Analysts can actively participate in the automated test case review process and give their feedback to enhance it.

### Domain-Driven Design

[Domain-Driven Design](https://en.wikipedia.org/wiki/Domain-driven_design) (DDD) is a software design approach that places the business domain at the center of the model. Instead of organizing code around technical layers alone, DDD encourages teams to express the system using the language of the problem space. This makes the code easier to understand, easier to change, and easier to test because the domain concepts remain explicit and consistent.

The core concepts in DDD can be summarized as:

- **Ubiquitous Language**: Use the same terminology in code, tests, and conversations that the business uses to describe the domain;
- **Bounded Contexts**: Separate large systems into explicit boundaries where terms, models, and rules are consistent within each context;
- **Entities**: Represent domain concepts that have identity and continuity over time;
- **Value Objects**: Represent descriptive concepts without identity, usually immutable and compared by value;
- **Aggregates**: Group related entities and value objects under a consistency boundary, with external access through the aggregate root;
- **Domain Services**: Capture important domain behavior that does not naturally belong to a single entity or value object;
- **Domain Events**: Represent meaningful business events that occurred in the domain and can trigger reactions across contexts.

DDD helps us focus on business behavior instead of implementation detail. Domain rules can be tested directly through entities, services, and value objects, while integration tests verify that boundaries between contexts and infrastructure behave correctly.

### Spec-Driven Development

[Spec-Driven Development](https://en.wikipedia.org/wiki/Specification-driven_development) (SDD) is an approach where the specification becomes the primary source of truth for implementation. The goal is to describe expected behavior clearly before the code is written, then use that specification to guide development and verification. This reduces ambiguity, aligns stakeholders, and makes it easier to keep requirements and tests in sync.

The process typically looks like this:

1. **Idea**: Identify the problem, user need, or opportunity that should be addressed;
2. **Spec**: Write a clear specification with examples, edge cases, and acceptance criteria;
3. **Review**: Align the specification with stakeholders, developers, and QAs before implementation;
4. **Tests**: Turn the approved specification into executable tests or reviewable test cases;
5. **Code**: Implement the smallest amount of code needed to satisfy the specification;
6. **Validation**: Run the test suite and confirm the implementation behaves as specified.
 
In testing terms, spec-driven development is useful because it gives the team a concrete contract to validate against. It works especially well when paired with BDD, acceptance criteria, and automated tests that verify behavior from the user’s perspective.

### Test-Driven Development

[Test-Driven Development](https://en.wikipedia.org/wiki/Test-driven_development) (TDD) is a testing methodology or a programming practice implemented from the developer’s perspective. The test is developed which specifies and validates what the code will do. In simple terms, test cases are created before code is written. The purpose of TDD is to make the code clearer, simple and bug-free. The simple concept of TDD is to write and correct the failed tests before writing new code. This helps to avoid duplication of code as we write a small amount of code at a time in order to pass tests.

Following steps define how to perform TDD:

1. Add a test.
2. Run all tests and see if any new test fails.
3. Write some code.
4. Run tests and see all pass.
5. Refactor code.
6. Repeat.

### Acceptance Test-Driven Development

In [Acceptance Test-Driven Development](https://en.wikipedia.org/wiki/Acceptance_test%E2%80%93driven_development) (ATDD) technique, a single acceptance test is written from the user’s perspective. It mainly focuses on satisfying the functional behavior of the system. This technique attempts to answer the question: *Is the code working as expected?* ATDD is very similar to BDD. However, a key difference between them is that BDD focuses more on the behavior of the feature, whereas ATDD focuses on capturing the accurate requirements.

This technique enhances collaboration among developers, users, and QAs with a common focus on defining the acceptance criteria. The following are some of the key practices in ATDD:

1. Analyzing and discussing the real-world scenarios;
2. Deciding the acceptance criteria for those scenarios;
3. Automating the acceptance test cases;
4. Focusing on the development of those requirement cases.

There are [key differences between TDD, BDD and ATDD](https://www.browserstack.com/guide/tdd-vs-bdd-vs-atdd). Understanding how these methods work can help developers and other individuals involved in software development figure out which strategy works best to achieve their goals.

## Testing Types

Software Testing Type is a classification of different testing activities into categories, each having, a defined test objective, test strategy, and test deliverables. We can define more than [100 types of Software Testing Types](https://www.guru99.com/types-of-software-testing.html). However, all testing types are not used in all types of projects.

Tests can be manual or automated. Both approaches are widely used and are paramount for the creation of a highly functional and robust application. By definition:

- *Manual testing* presupposes that developers/testers manually check the software, comparing the expected and actual results and detecting issues. Testers use the application simulating the real end users’ behavior, watching how it is working from their side.
- *Automated testing* is performed by the automated program algorithms that repeat preset actions. In this way, the program compares expected and actual outcomes and gives signals of possible defects.

In general, the most important automated test types for web applications are:

- **Unit Tests**: Testing of individual functions or classes by supplying input and making sure the output is as expected;
- **Integration Tests**: Verifying that multiple components, services, or modules work together correctly, including their interactions and side effects;
- **Functional Tests**: Testing scenarios on the product itself, by controlling the browser or the website, regardless of the internal structure to ensure expected behavior;
- **Visual Tests**: Detecting and reviewing visual changes to ensure the websites and applications always look exactly as intended.

Each type of test has an important part to play. Unit tests for fast developer feedback, integration tests to cover all the corner cases of component integrations, and functional tests to make sure everything works right for the end users.

## Testing Pyramid

[Testing Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html) is a metaphor that groups software tests into layers of different granularity. It also provides guidance on how many tests we should have in each layer. Essentially, the testing pyramid, also referred to as the [Test Automation Pyramid](https://www.browserstack.com/guide/testing-pyramid-for-test-automation), lays out the types of tests that should be included in an automated test suite. It also outlines the sequence and frequency of these tests. The goal is to provide immediate feedback and ensure that code changes do not disrupt existing features.

![](data:image/jpeg;base64,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)

Fig. 1: Testing Pyramid

Unit tests form the base of the testing pyramid. They test individual components or functions to validate that they work as expected in isolation. It is important to cover several scenarios in unit tests, for example, happy paths and error handling. Since this is the largest subset, the unit test suite should run as quickly as possible. Integration tests are the second layer of the testing pyramid. Fundamentally, they verify how a feature communicates with external dependencies. They run more slowly than unit tests. At the top of the pyramid are end-to-end tests. These ensure that the entire application functions as required. End-to-end tests sit at the top of the pyramid because they usually take the longest to run. They can also be fragile because they must cover a wide variety of user scenarios.

As we move up the Testing Pyramid, tests become more expensive because they are harder to create and maintain, and they are often more flaky because they rely on fragile interfaces. At the very top are manual tests, which still have their place, even in the modern age of automation and DevOps, because validating a new feature for the first time requires a human to decide whether it meets the [acceptance criteria](https://www.altexsoft.com/blog/business/acceptance-criteria-purposes-formats-and-best-practices/) (typically validated during User Acceptance Testing, UAT).

The testing pyramid helps address the problem of over-reliance on long-running UI tests. However, there are two known anti-patterns. These test suites look like [an “ice cream cone” and an “hourglass”](https://medium.com/@timothy.cochran/test-pyramid-the-key-to-good-automated-test-strategy-9f3d7e3c02d5). An ice cream cone has very few unit tests, some UI tests, lots of QA tests, and many manual tests. An hourglass has many unit and UI tests but few integration tests. Both patterns are problematic because heavy UI coverage makes the regression suite slow. This affects the productivity of the whole team.

In an Angular context, there are several types of tests: isolated and shallow unit tests, integration tests between components, and e2e tests, which can be split into functional and visual tests. Based on [Complete Guide to Angular Testing](https://christianlydemann.com/complete-guide-to-angular-testing/), a common recommendation is that 80% of tests should be unit tests, 15% integration tests, and 5% UI tests. These percentages may vary, but the ratio is usually similar. However, other opinions recommend thinking in [scales instead of pyramids](https://testguild.com/testing-pyramid/). Rather than prescribing fixed amounts of each test type, we should ask, “What risks exist in my system? How can my system break?” and then build the coverage needed in each area to address those risks.

## Testing Tools

Choosing the right testing tools depends on the type of feedback we need, the level of confidence we want to achieve, and how quickly the test suite must run. In Angular applications, the tooling typically spans unit and integration testing, test-data and dependency mocking, and full end-to-end browser automation. Angular recommends [Vitest testing framework](https://vitest.dev/) for unit and integration tests. For e2e tests, [Playwright](https://playwright.dev/) is recommended. However, we also have other available tools with higher popularity.

In the next table we have a combination of Angular recommendations and the strongest tools:

| Type of Tests | Type of Solutions | Tools |
| --- | --- | --- |
| Unit and Integration Tests | Recommended | [jest](https://www.npmjs.com/package/jest)<br>[rxjs-marbles](https://www.npmjs.com/package/rxjs-marbles) |
|  | Alternative | [vitest](https://www.npmjs.com/package/vitest) |
|  | Boilerplate code | [ng-mocks](https://www.npmjs.com/package/ng-mocks)<br>[@ngneat/spectator](https://www.npmjs.com/package/@ngneat/spectator)<br>[ts-mockery](https://www.npmjs.com/package/ts-mockery) |
|  | API mocking | [angular-in-memory-web-api](https://www.npmjs.com/package/angular-in-memory-web-api) |
|  | Reports | [istanbul-reports](https://www.npmjs.com/package/istanbul-reports)<br>[jest-html-reporters](https://www.npmjs.com/package/jest-html-reporters) |
| Functional and Visual Tests | Recommended | [playwright](https://www.npmjs.com/package/playwright)<br>[cucumber](https://www.npmjs.com/package/@cucumber/cucumber) |
|  | Alternative | [cypress](https://www.npmjs.com/package/cypress)<br>[puppeteer](https://www.npmjs.com/package/puppeteer)<br>[jest-image-snapshot](https://www.npmjs.com/package/jest-image-snapshot) |
|  | API mocking | [ng-apimock](https://www.npmjs.com/package/ng-apimock) |
|  | Reports | [allure-playwright](https://www.npmjs.com/package/allure-playwright)<br>[playwright-html-reporter](https://www.npmjs.com/package/playwright-html-reporter) |

In unit and integration tests, we can choose [Jest](https://www.npmjs.com/package/jest) or [Vitest testing framework](https://vitest.dev/). Both solutions are good choices, but we have other JavaScript test frameworks. Testing with [Mocha](https://www.npmjs.com/package/mocha) and [Chai](https://www.npmjs.com/package/chai) is an excellent alternative. Another solution that has been growing is [Testing Library](https://testing-library.com/). The library provides special tools for different frameworks like [React](https://www.npmjs.com/package/react), [Preact](https://www.npmjs.com/package/preact), [React Native](https://www.npmjs.com/package/react-native), [Angular](https://www.npmjs.com/package/@angular/core), [Vue](https://www.npmjs.com/package/vue), and others. It also helps with e2e testing tools like [Cypress](https://www.npmjs.com/package/cypress), [Puppeteer](https://www.npmjs.com/package/puppeteer), [WebdriverIO](https://www.npmjs.com/package/webdriverio), and others. For testing observables, [marble diagrams](https://rxmarbles.com/) are recommended.

To test Angular components, it is important to reduce boilerplate code. [ngMocks](https://www.npmjs.com/package/ng-mocks) is indispensable in Angular projects for creating mocks for tests. [Spectator](https://www.npmjs.com/package/@ngneat/spectator) is another library to simplify the Angular tests. This library is recommended only for testing templates. It happens that in many cases we test the same thing with functional tests. To avoid duplicate effort, HTML is most often tested with functional tests.

[Cypress](https://www.cypress.io/) or [Jest](https://jestjs.io/) with [Puppeteer](https://www.npmjs.com/package/puppeteer) are very popular testing solutions for e2e tests. [Cucumber](https://cucumber.io/) is a framework that runs automated acceptance tests written in a BDD style. Cypress is quite different technology for different tasks from Playwright. It works well for testing components of a web application, but it cannot replace Playwright. This means that a better combination is Jest + Cucumber + Playwright. Popular solutions for API mocking are [ng-apimock](https://www.npmjs.com/package/ng-apimock), [json-server](https://www.npmjs.com/package/json-server) and [mockserver](https://www.npmjs.com/package/mockserver).

The previous table includes both recommended and alternative solutions. Libraries from both proposals can be combined, but equivalent libraries should not be mixed within the same project. For example, if Jest is selected for unit testing, it should also be used for integration testing. Avoid using Jest for one test suite and, for example, Vitest for another when they fulfill the same role.

## Unit Tests

### Fundamentals

[Unit Testing](https://en.wikipedia.org/wiki/Unit_testing) is a software testing method in which individual units or components of a software application (such as functions, methods, or classes) are tested in isolation to verify that they work correctly as expected.

In the testing pyramid, unit tests are the base layer and should provide the fastest feedback loop for developers. In practical Angular work, a unit can be a method in a service, a pipe transform, a validator, or a focused behavior in a component class. The main goal is to verify one behavior at a time with deterministic inputs and outputs. To keep this feedback fast and deterministic, unit tests should isolate the code under test from external dependencies. A component under test does not need real services, HTTP calls, timers, or browser integrations. Instead, we use simplified objects that mimic production behavior only where needed. This reduces complexity, avoids flaky tests, and makes failures easier to diagnose. A [Test Double](http://xunitpatterns.com/Test%20Double.html) is the generic term for these simplified objects. Although test doubles come in many flavors, teams often use the term "mock" informally to refer to most of them.

There are five types of doubles:

- **Dummy Object**: Dummy objects are passed around but never actually used. Usually they are just used to fill parameter lists.
- **Fake Object**: Fakes are objects that have working implementations, but not the same as the production implementation. Usually they take some shortcut and have simplified version of production code.
- **Test Stub**: Stubs provide canned answers to calls made during the test, usually not responding at all to anything outside what's programmed in for the test.
- **Test Spy**: Spies are stubs that also record some information based on how they were called. One form of this might be an email service that records how many messages it was sent.
- **Mock Object**: Mocks are pre-programmed with expectations which form a specification of the calls they are expected to receive. They can throw an exception if they receive a call they don't expect and are checked during verification to ensure they got all the calls they were expecting.

[Vitest](https://vitest.dev/) has test double functions called spies. A spy can stub any function and track calls to it and all arguments. A spy only exists in the `describe` or `it` block in which it is defined and will be removed after each test. There are special matchers for interacting with spies. Jest and Vitest provide similar resources for mocking. In both frameworks, [mocks](https://vitest.dev/api/vi.html#vi-mockobject-3-2-0) are used to stub out functionality with mock data.

Angular has been designed with testability as a primary goal. In practice, unit testing in Angular combines framework helpers with general testing capabilities from Jest or Vitest. This foundation leads directly into the next sections, where we cover isolated tests, shallow tests, asynchronous behavior, and other patterns used in day-to-day Angular projects.

To learn more about Angular unit tests, [Angular Testing](https://angular.dev/guide/testing), [Testing Angular Applications](https://www.manning.com/books/testing-angular-applications) and [Angular Test-Driven Development](https://www.packtpub.com/en-pt/product/mastering-angular-test-driven-development-9781805127932) books, [Testing Angular](https://testing-angular.com/) online book, and articles such as [The Complete Guide to Angular Testing](https://christianlydemann.com/complete-guide-to-angular-testing/), [Angular Testing Recipes](https://github.com/juristr/angular-testing-recipes), and [Three Ways to Test Angular Components](https://vsavkin.com/three-ways-to-test-angular-2-components-dcea8e90bd8d) are good references.

### Isolated Tests

Isolated tests are used to test a unit in isolation. The unit might contain some business logic that needs to be tested in isolation. When writing isolated unit tests all external dependencies should be mocked. The most common examples of isolated tests are tests of functions and class methods. In Angular applications, we should prefer isolated tests to test Services and Pipes.

We can use isolated tests or shallow tests to test Angular components, whichever makes the most sense. With isolated tests, we only will test the class of the component. It is often useful to test complex components without rendering them. To see how it can be done, see the following component:

```typescript
@Component({
  selector: 'cmn-view-header',
  template: `
    <h1>My Component</h1>
    <cmn-button type="primary-button" (clicked)="onClick()"></cmn-button>
  `,
})
export class ViewHeaderComponent {
  constructor(private readonly sharedService: SharedService) {}

  onClick(): void {
    this.sharedService.copyToClipBoard();
  }
}
```

In the example, we have a component that calls a service when we click on the button. An example of an isolated test:

```typescript
describe('ViewHeaderComponent', () => {
  let component: ViewHeaderComponent;
  let sharedService: SharedService;

  beforeEach(() => {
    sharedService = { copyToClipBoard: jest.fn() } as unknown as SharedService;
    component = new ViewHeaderComponent(sharedService);
  });

  it('#onClick should call #copyToClipBoard', () => {
    component.onClick();

    expect(sharedService.copyToClipBoard).toHaveBeenCalled();
  });
});
```

As we can see, testing Angular components in isolation is no different from testing any other JavaScript object. This approach is preferable when other types of tests are used to interact with the DOM.

### Shallow Tests

Testing component classes without rendering their templates works in certain scenarios, but not in all of them. Sometimes, we can only write a meaningful test if we render a component's template. However, we want to keep the test isolated. In this case, we only need to render the template without rendering the child components. This is what is colloquially known as shallow test.

An example of a shallow test:

```typescript
describe('ViewHeaderComponent', () => {
  let button: ButtonComponent;
  let component: ViewHeaderComponent;
  let fixture: ComponentFixture<ViewHeaderComponent>;
  let sharedService: SharedService;

  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [ViewHeaderComponent, MockComponent(ButtonComponent)],
      providers: [
        { provide: SharedService, useValue: { copyToClipBoard: jest.fn() } }
      ],
    }).compileComponents();

    sharedService = TestBed.inject(CommonSharedService);
    fixture = TestBed.createComponent(ViewHeaderComponent);
    component = fixture.componentInstance;
  });

  it('should have <h1> with "My Component"', () => {
    const h1 = fixture.nativeElement.querySelector('h1');

    expect(h1.textContent).toEqual('My Component');
  });

  it('#onClick should call #copyToClipBoard', () => {
    button = fixture.debugElement.query(By.directive(ButtonComponent)).componentInstance;
    button.clicked.emit();

    expect(sharedService.copyToClipBoard).toHaveBeenCalled();
  });
});
```

In this example, all external dependencies are mocked (services and components). To mock components, we can use the [ng-mocks](https://www.npmjs.com/package/ng-mocks) library. Unlike [NO_ERRORS_SCHEMA](https://angular.dev/api/core/NO_ERRORS_SCHEMA), which can hide certain issues and does not help with configuring inputs and outputs, [ng-mocks](https://www.npmjs.com/package/ng-mocks) automatically mocks component inputs, outputs, and other bindings for us. The NO_ERRORS_SCHEMA tells the Angular compiler to ignore unrecognized elements and attributes. The NO_ERRORS_SCHEMA also prevents the compiler from telling us about the missing components and attributes that we omitted inadvertently or misspelled. We could waste hours chasing phantom bugs that the compiler would have caught in an instant.

The most common issue in shallow testing comes from the amount of boilerplate code. We can create helpers and factory functions to reduce the boilerplate code. Alternatively, we can use [Spectator](https://www.npmjs.com/package/@ngneat/spectator) to simplify Angular tests.

### Snapshot Tests

Snapshot tests are a valuable way to ensure that a UI does not change unexpectedly. In a typical snapshot test for a web application, a UI component is rendered, a snapshot is captured, and the result is compared against a reference snapshot stored alongside the test. The test fails if the snapshots differ, indicating either an unintended change to the UI or that the reference snapshot should be updated to reflect the component's new expected state.

Snapshot testing are the best alternative to the [DOM testing](https://angular.dev/guide/testing/components-basics#component-dom-testing). Rather than asserting values obtained from specific `HTMLElement` instances, we can validate the entire component fixture against a snapshot. This approach requires updating our tests to validate components against snapshots rather than relying on traditional DOM assertions.

An example of a snapshot test:

```typescript
describe("ViewHeaderComponent", () => {
  let component: ViewHeaderComponent;
  let fixture: ComponentFixture<ViewHeaderComponent>;

  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [ViewHeaderComponent, MockComponent(ButtonComponent)],
      providers: [{ provide: SharedService, useValue: {} }],
    }).compileComponents();

    fixture = TestBed.createComponent(ViewHeaderComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should render <h1> with "My Component"', () => {
    expect(fixture).toMatchSnapshot();
  });
});
```

Note that in the initial run (when there aren’t any snapshots to test against), Jest or Vitest will treat the first output as the main reference and will save the output in a *__snapshots__* directory in the same directory as the test file. Next example shows the *.snap* file generated by Jest:

```html
exports[`ViewHeaderComponent should render <h1> with "My Component" 1`] = `
<cmn-view-header>
  <h1>
    My Component
  </h1><cmn-button
    type="primary-button"
  />
</cmn-view-header>
`;
```

Snapshot testing is not limited to HTML templates. It can also be used to validate JavaScript objects, function outputs, JSON data, configuration files, and many other types of content.

Snapshot tests once promised a revolution, but today [they are often criticized](https://medium.com/@tomgold_48918/why-i-stopped-using-snapshot-testing-with-jest-3279fe41ffb2) for being fragile, creating a false sense of safety, and losing value over time. Even small code changes can cause snapshot failures. At first, teams review those failures carefully and update snapshots when needed. Over time, however, people may start updating snapshots automatically without reviewing them, assuming everything is fine. When that happens, snapshots stop providing meaningful protection, and real regressions can slip into production. Snapshot testing is not inherently bad, but it often provides limited value for template-level UI verification. It is usually more useful for stable outputs such as JSON payloads, configuration files, and large JavaScript objects.

### Marble Tests

We can test asynchronous [RxJS](https://rxjs-dev.firebaseapp.com/) code synchronously and deterministically by virtualizing time with the [TestScheduler](https://rxjs-dev.firebaseapp.com/api/testing/TestScheduler). When we need to test Observables, there are [two main patterns](https://medium.com/angular-in-depth/how-to-test-observables-a00038c7faad): the *subscribe and assert pattern* and the *marble testing pattern* based on diagrams. A marble diagram is a domain-specific language in RxJS that helps us model the interactions and values of one or more observables in a test. It is a visual representation, like a schema, that describes what is happening in an observable stream.

We have helper methods that have been added to make creating marble tests easier:

- `hot(marbles: string, values?: object, error?: any)` creates a "hot" observable (a subject) that will behave as though it's already "running" when the test begins. An interesting difference is that *hot* marbles allow a *^* character to signal where the "zero frame" is. That is the point at which the subscription to observables being tested begins.
- `cold(marbles: string, values?: object, error?: any)` creates a "cold" observable whose subscription starts when the test begins.
- `expectObservable(actual: Observable<T>).toBe(marbles: string, values?: object, error?: any)` schedules an assertion for when the *TestScheduler* flushes. The *TestScheduler* will automatically flush at the end of *it* block.
- `expectSubscriptions(actual: SubscriptionLog[]).toBe(marbles: string)` like `expectObservable` schedules an assertion for when the *testScheduler* flushes. Both `cold()` and `hot()` return an observable with a property `subscriptions` of type `SubscriptionLog[]`. Give subscriptions as parameter to `expectSubscriptions` to assert whether it matches the subscriptions marbles diagram given in `toBe()`. Subscription marble diagrams are slightly different than Observable marble diagrams.

To write a test with marble diagrams we will need to stick to a convention of characters that will help visualize the observable stream:

- ' ' the whitespace is a unique character that will not be interpreted;
- '-' represents a frame of virtual time passing;
- '|' this sign illustrates the completion of an observable;
- '#' signifies an error;
- [a-z] an alphanumeric character represents a value which is emitted by the Observable;
- '()' used to group events in the same frame. This can be used to group values, errors, and completion;
- '^' this sign illustrates the subscription point and is used when we are dealing with hot observables;
- '!' this sign indicates the end of a subscription point.

Most [marble testing](https://rxjs-dev.firebaseapp.com/guide/testing/marble-testing) documentation is available in the official RxJS docs. There are also different libraries that help us write marble tests. For Jest-based projects, the most common choices are “[jest-marbles](https://www.npmjs.com/package/jest-marbles)” and “[rxjs-marbles](https://www.npmjs.com/package/rxjs-marbles)”. RxJS itself also provides testing utilities, and these libraries are convenience wrappers around them. This syntax gives us the power to create and manipulate observable streams.

For example, for the following observable:

```typescript
export const source: Observable<number> = of(1, 2, 3).pipe(map(x => x * 2));
```

The marble test can be written as follows:

```typescript
describe('Observable', () => {
  it('should multiply by "2" each value emitted', () => {
    const expected = cold('(abc|)', { a: 2, b: 4, c: 6 });

    expect(source).toBeObservable(expected);
  });
});
```

Notice the *source* and *expected* variable declarations. In marble testing, we use this pattern frequently. With this pattern, we can see when values (a, b, c) are emitted, when the stream completes (|), and compare the actual output directly with what is *expected*.

### Asynchronous Tests

JavaScript code frequently runs asynchronously. Whether we are fetching data, reading files, or waiting on timers, the framework needs to know when the code it is testing has completed before moving on to the next test. Here are the patterns we will use most often. Testing asynchronous code involves testing Observables, Promises and Timers.

The native timer functions (i.e., *setTimeout*, *setInterval*, *clearTimeout*, *clearInterval*) are less than ideal for a testing environment since they depend on real time to elapse. [Vitest timers](https://vitest.dev/guide/mocking/timers) and [Jest Timer Mocks](https://jestjs.io/docs/timer-mocks) can swap out timers with functions that allow us to control the passage of time.

To see how it can be done, see the following class:

```typescript
export class TimerService {
  started = false;

  startTimer(): void {
    setTimeout(() => { this.started = true; }, 100);
  }
}
```

The unit test can be written as follows:

```typescript
describe('TimerService', () => {
  let service: TimerService;

  beforeEach(() => {
    service = new TimerService();
    jest.useFakeTimers();
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  it('should update #started after timeout', () => {
    service.startTimer();
    jest.advanceTimersByTime(100);

    expect(service.started).toBe(true);
  });
});
```

Alternatively, Angular provides helper functions [*fakeAsync*](https://angular.dev/api/core/testing/fakeAsync) and [*tick*](https://angular.dev/api/core/testing/tick) to handle asynchronous tests, which essentially gives us an easy way to run asynchronous code before the assertions. When a test is running within a [*fakeAsync*](https://angular.dev/api/core/testing/fakeAsync) zone, we can use two functions called [*flushMicrotasks*](https://angular.dev/api/core/testing/flushMicrotasks) and [*tick*](https://angular.dev/api/core/testing/tick). The *tick* function will advance time by a specified number of milliseconds, so *tick(100)* would execute any asynchronous tasks that would occur within 100ms. The *flushMicrotasks* function will clear any “microtasks” that are currently in the queue.

The *fakeAsync* utility function has a few limitations. It won't work if the test body makes an *XMLHttpRequest* (XHR) call. XHR calls within a test are rare so we can generally stick with *fakeAsync*. But if we ever do need to call *XMLHttpRequest*, we'll use the [*waitForAsync*](https://angular.dev/api/core/testing/waitForAsync) utility function.

When we need to test RxJS asynchronous code, marble testing can be an elegant way to test it. RxJS provides the [TestScheduler](https://rxjs.dev/api/testing/TestScheduler) which has a `run` callback. The *TestScheduler* class is part of the `rxjs/testing` module, and enables synchronous testing of asynchronous, or synchronous, Observables.

In the next example, we can see how *TestScheduler* works:

```typescript
export const source: Observable<number> = of(1, 2, 3).pipe(delay(100), map(x => x + 1));
```

The marble test can be written as follows:

```typescript
describe('TestScheduler', () => {
  let scheduler: TestScheduler;

  beforeEach(() => {
    scheduler = new TestScheduler((actual, expected) => {
      expect(actual).toEqual(expected);
    });
  });

  it('should wait 100ms before emitting values', () => {
    scheduler.run(({ expectObservable }) => {
      expectObservable(source).toBe('100ms (abc|)', { a: 2, b: 3, c: 4 });
    });
  });
});
```

In marble tests, time should be expressed using the *TestScheduler*, not real milliseconds.

### Code Coverage

Code coverage is a measurement of the percentage of source code executed during automated tests. Test coverage, on the other hand, measures the effectiveness of automated test cases in a qualitative manner.

We can configure minimum coverage thresholds. Thresholds specified as positive numbers represent the minimum required percentage. Anything below these thresholds will cause the test suite to fail with an exit code of 1. Thresholds specified as negative numbers represent the maximum number of allowed uncovered entities. Thresholds can be specified as “global” if we want them to be applied to every file in the project, or as a path or glob if we only want to apply them to certain files.

For example, the following configuration causes test failure if global coverage falls below 80% for statements, lines, branches, and functions:

```javascript
{
  thresholds: {
    global: {
      statements: 80,
      lines: 80,
      branches: 80,
      functions: 80
    }
  }
}
```

This configuration works with Vitest and Jest. For projects with Jest, more details are available in the [coverageThreshold](https://jestjs.io/docs/en/configuration#coveragethreshold-object) section. For Vitest, coverage configuration is available in the [Vitest coverage documentation](https://vitest.dev/guide/coverage.html).

While line and branch coverage indicate which code paths are exercised, they do not prove that tests actually detect incorrect behavior. [Mutation testing](https://en.wikipedia.org/wiki/Mutation_testing) addresses this gap by automatically introducing small changes, called mutants, into the production code and then verifying that at least one test fails for each change. A mutant that is not caught by any test is called a survivor and represents a gap in the test suite. The ratio of killed mutants to total mutants is the mutation score, which is a more meaningful quality indicator than line coverage alone.

[Stryker Mutator](https://stryker-mutator.io/) is the leading mutation testing tool for JavaScript and TypeScript projects, with dedicated support for Angular via [@stryker-mutator/vitest-runner](https://www.npmjs.com/package/@stryker-mutator/vitest-runner) and [@stryker-mutator/jest-runner](https://www.npmjs.com/package/@stryker-mutator/jest-runner). Running mutation tests on critical business logic and domain services helps identify weak assertions and untested edge cases that coverage metrics alone would not surface.

### Test Reporters

It’s very useful to know which parts of the code are not covered by unit tests. In Angular projects, coverage data can be captured and reported by passing the *--code-coverage* flag. Vitest and Jest rely on [Istanbul](https://istanbul.js.org) to generate code coverage reports. By default, these tools report in multiple formats (*json*, *lcov*, *text*, *clover*), but we can configure them to use any valid Istanbul reporter. Running coverage with an *html* report creates a visual overview of coverage in HTML format. We can view results for the entire app broken down per file and examine individual files to see how often each line was executed. Using the *lcov* report as part of the CI pipeline enables us to track coverage trends across builds.

### Debugging

Relying only on *console.log* is rarely enough when a test fails intermittently or only in CI. A stronger approach is to debug tests with breakpoints, source maps, and deterministic test execution.

For Angular tests running in a browser-like environment, the first step is usually to run a single spec in watch mode and attach debugger tools. This allows us to inspect component state, dependency injections, async queues, and DOM updates frame by frame. The Angular docs provide a practical baseline in [Debugging Tests](https://angular.dev/guide/testing/debugging).

A common workflow is to debug one test file in a single worker, then attach Node DevTools. The following npm script shows the default script for Jest:

```json
{
  "scripts": {
    "test:debug": "node --inspect-brk node_modules/jest/bin/jest.js --runInBand"
  }
}
```

An equivalent npm script can be used for Vitest. In both frameworks, `debugger` statements inside tests or production code under test pause execution when the inspector is attached.

In day-to-day development, it is usually better to prefer IDE test plugins over raw terminal commands for run/debug loops. In VS Code, extensions such as [Vitest Extension](https://marketplace.visualstudio.com/items?itemName=vitest.explorer) and [Jest Extension](https://marketplace.visualstudio.com/items?itemName=Orta.vscode-jest) integrate with the Testing panel, where we can run a single test, debug directly from `it`/`describe` blocks, inspect stack traces, re-run failed tests, and navigate quickly between failures and source code. This shortens feedback cycles and reduces context switching.

### Best Practices

The following practices summarize how to design unit tests that are deterministic, readable, and cost-effective to maintain:

- [ ] **Follow conventions**: A style guide is a set of standards for how code should be written and organized. Good projects have clear, consistent coding conventions with automated enforcement. Conventions must be dynamic and adaptable for each team and project, and each team defines its convention. Some guides such as [A guide to unit testing in JavaScript](https://github.com/mawrkus/js-unit-testing-guide) and [Node.js & JavaScript Testing Best Practices](https://github.com/goldbergyoni/javascript-testing-best-practices), and [Frontend testing standards and style guidelines](https://docs.gitlab.com/ee/development/testing_guide/frontend_testing.html) can be used as a reference.

- [ ] **Follow Design Principles**: The key to good unit testing is writing testable code. Use good naming and comments, avoid duplication, follow single responsibility, keep one behaviour in one test, ensure test order does not affect other tests, minimize dependencies, use design patterns, and avoid global mutable state.

- [ ] **Follow the Arrange-Act-Assert pattern**: The Arrange-Act-Assert (AAA) pattern separates test setup, test action, and test assertion. Arrange creates objects and mocks and sets expectations, Act invokes the method, and Assert verifies outcomes. AAA is well known and recommended, and explicit comments for each section are usually unnecessary. See next example in Jasmine style:

  ```javascript
  describe('Array', () => {
    describe('.push(x)', () => {
      let initialArray;

      beforeEach(() => {
        initialArray = [1]; // Arrange
        initialArray.push(2); // Act
      });

      it('appends x to the end of the Array', () => {
        expect(initialArray).toEqual([1, 2]); // Assert
      });
    });
  });
  ```

- [ ] **Follow Test-Driven Development**: [Test-Driven Development](https://en.wikipedia.org/wiki/Test-driven_development) is a robust way to design units so behaviour is specified through tests. Consider writing tests first in `red-green-refactor` style. Test-first improves testable design, keeps code minimal, supports safe refactoring, gives fast feedback, and increases confidence to add features, fix bugs, and explore design changes.

- [ ] **Follow the FIRST Principles**: [FIRST](https://howtodoinjava.com/best-practices/first-principles-for-good-tests/) means tests should be Fast, Isolated, Repeatable, Self-validating, and Timely. Fast tests avoid slowing development/deployment, isolated tests avoid dependence on each other, repeatable tests produce consistent results, self-validating tests require no manual interpretation, and timely tests can be written early and often.

- [ ] **Use lint rules**: Linters catch severe issues and enforce consistency/readability. Beyond base ESLint style rules, include specialized plugins such as promise/security/jest/jasmine plugins for missing resolves, risky regex, missing assertions, and skipped tests.

- [ ] **Describe expectations in a human language**: Name `describe` and `it` blocks so intention is clear in human language. The full test name is the `it` title plus parent `describe` titles, so wording matters. Prefer assertive verbs and avoid vague terms like “should.” This makes test suites useful documentation and produces meaningful failure messages.

- [ ] **Name the tests properly**: Test names should be concise, explicit, descriptive, and written in correct English. Read the spec runner output and ensure it is understandable for others. Test names can become live documentation for the codebase.

- [ ] **Structure the tests properly**: Nest suites when needed to structure tests logically into subsets. This helps tests build from least to most specific, makes extension/refactor easier, reduces repetition in branch testing, and encapsulates tests by common denominator. See next example:

  ```javascript
  describe('A set of functionalities', () => {
    describe('A subset of functionalities', () => {
      it('does something great', () => {});

      it('does something awesome', () => {});
    });

    describe('Another subset of functionalities', () => {
      it('also do something great', () => {});
    });
  });
  ```

- [ ] **Categorize tests under at least 2 levels**: Structure suites with at least two `describe` levels: one for the unit under test and one for scenario/category. This improves report readability and navigation in larger suites.

- [ ] **Create new tests for every defect**: Add regression tests for bug fixes so defects do not recur. Start by creating a test that reproduces the issue before changing code, then apply TDD to fix it. This improves quality assurance and reduces post-release glitches.

- [ ] **Mock Dates and Timers**: When production code uses `new Date()` or `Date.now()`, mock time for deterministic tests. Also, Timers (`setTimeout`, `setInterval`, etc.) run normally if we don't mock them.

- [ ] **Mock out all external services and state**: Unit tests should be stateless and should not modify external resources. Otherwise shared external state can make tests influence each other. Mocking external services also makes tests run much faster than using real-time dependencies.

- [ ] **Do not test external libraries**: Do not test library internals, but rely on library test coverage. Testing thin wrappers that only call library functions usually adds little value. Prefer testing behaviour from the user interaction perspective.

- [ ] **Do not mock everything**: Unit tests should mock external side effects, but not every dependency must be mocked. Real objects are acceptable when setup stays simple, no shared state appears, no AJAX/API/reload side effects happen, and execution speed remains within limits.

- [ ] **Do not test multiple concerns**: If a method has multiple end results, test each result separately. Tests combining concerns (for example with “AND” or “OR” in names) are a smell. Single-concern tests improve failure diagnosis. See next bad example:

  ```javascript
  it('sends the profile to the server and updates the profile view', () => {
    // expect(...).to(...);
    // expect(...).to(...);
  });
  ```

  Where is better changing to:

  ```javascript
  it('sends the profile to the server', () => {
    // expect(...).to(...);
  });

  it('updates the profile view', () => {
    // expect(...).to(...);
  });
  ```

- [ ] **Do not skip unit tests**: Do not comment tests out because they are slow, complex, or flaky. Make tests fast, simple, and trustworthy, otherwise remove them. Avoid `xdescribe`/`xit` and enforce `no-disabled-tests` and `no-focused-tests` lint rules.

- [ ] **Do not test configuration settings**: Configuration values are not usually a unit of business logic. Keep only one or two tests for configuration loading itself instead of many copy-paste tests for each setting. Snapshot testing can verify full configuration content efficiently.

- [ ] **Do not overuse variables**: Extracting too many values into variables can bloat tests and make failures harder to trace. In short tests, repeated literals may be clearer than excessive indirection.

- [ ] **Do not use static access**: Static access is an anti-pattern because it hides dependencies and side effects and harms testability. Prefer non-static methods, instantiate classes, and inject dependencies via constructor so tests can swap mocks or have alternative implementations.

- [ ] **Do not reuse production code**: Tests should verify production code, not reuse it heavily. Reusing production helpers in tests can hide bugs in those helpers. Prefer clear input-output assertions with hard-coded expected values.

- [ ] **Do not catch errors but expect them**: Avoid verbose try-catch assertions for expected failures. Prefer dedicated one-line error assertions like `toThrow`. Also validate useful error properties or types so failure handling is actionable. See next example:

  ```javascript
  it('adds a user in memory', () => {
    expect(userManager.addUser()).toThrow(new Error());
  });
  ```

- [ ] **Do not test complex user interactions**: Some scenarios are too broad for unit tests, such as multi-step drag-and-drop flows or gallery navigation across loaded data. Test these at higher levels (functional/integration). Keep unit tests focused on simple user actions and event-to-method behaviour.

- [ ] **Do not Repeat Yourself**: Use `before/after` hooks to DRY repeated setup, teardown, and actions. This keeps suites concise/readable and centralizes change points.

- [ ] **Avoid toBeTruthy or toBeFalsy**: Avoid broad truthy/falsy matchers because they can create weak or false-positive tests (`null` vs `false`, etc.). Prefer specific assertions for exact values and types.

- [ ] **Prefer toBe over toEqual for primitive values**: `toBe` uses `Object.is` and is generally faster and more precise for primitives. Use `toEqual` when deep comparison of objects/arrays is required.

- [ ] **Avoid the All's**: Prefer `beforeEach/afterEach` over `beforeAll/afterAll`. `All` hooks are not reset between tests and can leak state. `Each` hooks enforce independence and make execution flow clearer.

- [ ] **Avoid mocks in favor spies**: Stub dependencies that create side effects (network, external IO) and use spies to verify interactions. This keeps tests isolated and focused while avoiding unnecessary heavy mocking.

- [ ] **Avoid global test data**: Create data per test and clean up after each test to guarantee a clean environment. This prevents inter-test dependencies. If setup cost is high, simplify test data rather than sharing mutable fixtures.

- [ ] **Avoid logic in the tests**: Tests should avoid loops, switches, and conditionals because logic inside tests creates additional bug paths and weakens one-concern clarity.

- [ ] **Cover the general case and the edge cases**: Strange behaviour often appears at edges. Tests should cover common flows and edge cases, and coverage/test-coverage metrics should be used to evaluate effectiveness of exercised paths.

- [ ] **Test only public methods**: Testing internals creates large maintenance overhead with little payoff. Verifying public behaviour implicitly tests private implementation and keeps tests resilient unless outputs are wrong.

- [ ] **Test the behaviour and not the implementation**: Assert outcomes of operations, not internal state details. This allows internal refactoring without forcing test rewrites and preserves behavioural confidence.

- [ ] **Test many input combinations**: Real systems receive many permutations of input. Avoid only happy-path samples and use random or property-based combinations (for example [fast-check](https://www.npmjs.com/package/fast-check) or [jsverify](https://www.npmjs.com/package/jsverify)) to expose hidden failures.

- [ ] **Use realistic input data**: Unrealistic test inputs miss production-like bugs. Use realistic and varied data to increase defect detection. Libraries like [Faker](https://www.npmjs.com/package/faker) can generate pseudo-real data similar to production forms and distributions.

- [ ] **Use factory functions**: Factory functions simplify object creation in tests, reduce setup duplication, and improve readability by expressing setup as a single call. They also provide flexible instance initialization and clean dependency injection patterns.

- [ ] **Use the prefixes "actual" and "expected"**: When using variables in equality assertions, prefix them with `actual` and `expected`. This improves readability, clarifies intent, and reduces accidental mix-ups in assertions.

- [ ] **Design for lean testing**: Test code is different from production code. Keep it simple, short, flat, and easy to read. Keep only as much testing abstraction as needed and prioritize agility and clarity.

- [ ] **Shorten the feedback loop**: Testing effectiveness depends on quick iterative loops: try, get feedback, refactor. Faster feedback enables more improvement cycles. Running CI-like checks locally can accelerate this loop significantly.

- [ ] **Tag the tests**: Different scenarios require different subsets of tests (smoke, IO-less, commit-time, PR-time full E2E). Tag tests with keywords so tooling can grep and run only relevant subsets, avoiding slow full runs for small changes.

- [ ] **Focus on test coverage**: Coverage indicates how many lines/branches/statements are reached by tests. Very low coverage is risky, but 100% can be costly and may overemphasize edge corners over critical paths. A common rule of thumb is to set a limit of around [80%](https://martinfowler.com/bliki/TestCoverage.html).

- [ ] **Analyse the test coverage report**: Coverage reports help reveal under-tested code areas where surprising behaviour can hide. If we do not know what is untested, we do not know where issues may originate.

- [ ] **Avoid duplicate coverage**: Do not duplicate feature coverage across test layers unnecessarily. If functionality is well-covered in unit tests, avoid repeating it in expensive UI tests. Keep UI tests for big-picture user stories spanning components/views.

- [ ] **Measure logical coverage**: Traditional line coverage can mislead by showing visited code without proving behaviour correctness. Use [mutation testing](https://en.wikipedia.org/wiki/Mutation_testing) to validate whether tests actually detect logical changes, not just execute lines.

- [ ] **Parallelize test execution**: Run multiple tests simultaneously to shorten total feedback time. Parallelism reduces waiting and helps teams resolve issues faster than strictly serial runs.

- [ ] **Know the testing framework API**: Strong knowledge of the testing framework API reduces test size and complexity and improves readability. Using built-in helpers (for example concise spy APIs) often produces clearer tests than verbose low-level assertions.

## Integration Tests

[Integration testing](https://en.wikipedia.org/wiki/Integration_testing) is when we test two or more components together. This makes sense for parts of the applications where the integration between components is important to test, such as a container/smart component with container/dumb components. With integration testing, we simply import the implementation of the dependencies of the unit under test even though we still should mock out [HTTPClient](https://angular.dev/api/common/http/HttpClient).

An example of integration testing is:

```typescript
describe('ViewHeaderComponent', () => {
  let buttonElement: HTMLElement;
  let fixture: ComponentFixture<ViewHeaderComponent>;
  let sharedService: SharedService;

  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [ViewHeaderComponent, ButtonComponent],
      providers: [
        { provide: SharedService, useValue: { copyToClipBoard: jest.fn() } }
      ],
    }).compileComponents();

    sharedService = TestBed.inject(SharedService);
    fixture = TestBed.createComponent(ViewHeaderComponent);
    buttonElement = fixture.nativeElement.querySelector('button');
  });

  it('should call #copyToClipBoard when the button is clicked', () => {
    buttonElement.click();

    expect(sharedService.copyToClipBoard).toHaveBeenCalled();
  });
});
```

In this example, when a user clicks the button component, an event is emitted to the parent component. We cannot mock the *ButtonComponent* because its event listeners are defined within it. In shallow tests, all child components are mocked and events are emitted by calling *@Output()* directly. In this example, the event originates from the *ButtonComponent*.

Although both shallow and integration tests render components, they differ significantly in nature. In the shallow test, we mocked every dependency of the component. In the integration test, we mocked only the *SharedService*. Shallow tests are isolated and, as a result, can be used to drive component design. Integration tests serve to verify correctness.

As discussed, integration tests verify that groups of components and services work together. Other common scenarios for integration tests include testing integrations with [Angular Router](https://angular.dev/guide/routing), [NgRx Effects](https://ngrx.io/guide/effects), or [Redux Beacon](https://rangle.gitbook.io/redux-beacon/). However, it is recommended to avoid testing complex behaviors using only unit and integration tests. Complex user interactions should be tested with functional tests.

## Functional Tests

### Fundamentals

[Functional testing](https://en.wikipedia.org/wiki/Functional_testing), also known as end-to-end (e2e) testing in the Angular ecosystem, verifies the behavior of the application as a whole from the user’s perspective. Unlike unit and integration tests, which validate isolated pieces of logic or component interactions, functional tests drive a real browser through real user scenarios, interacting with the application as a human would. They sit at the top of the Testing Pyramid and are the most expensive to write and maintain, but they provide the highest level of confidence that the product works correctly in a production-like environment.

[Playwright](https://playwright.dev/) is an open-source framework developed by Microsoft for automating web browsers to test web applications. It simulates user interactions such as clicking, filling out forms, navigating between routes, and validating rendered state. Its features include headless execution, cross-browser support (Chromium, Firefox, and WebKit), mobile emulation, network interception, and advanced debugging tools, making it ideal for web applications.

The next example shows a purchase-order-style scenario where we validate the total number of promotions in the first section and the name of the first promotion.

```typescript
describe('Promotional Sections', () => {
  beforeEach(async ({ page }) => {
    await page.goto('/promotions');
  });

  it('should display promotions in the first section', async ({ promotionalSectionPO }) => {
    const firstSection = promotionalSectionPO.getAllSections().first();
    const promotions = promotionalSectionPO.getAllPromotions(firstSection);

    await expect(promotions).toHaveCount(2);
    await expect(promotionalSectionPO.getPromotionName(promotions.first())).toHaveText('Summer Sale');
  });
});
```

In Angular, functional tests are typically run against the compiled application served in a browser, covering flows that span multiple components, routes, services, and network calls. They are the right tool to validate critical user journeys such as login, checkout, or multi-step submissions.

### Browser Interaction

Functional tests automate real browser interactions, namely, navigating to URLs, clicking elements, filling in forms, selecting options, scrolling, hovering, pressing keyboard shortcuts, and verifying visible content or application state. The test framework controls the browser through browser automation protocols. Playwright does not use the [WebDriver](https://www.w3.org/TR/webdriver/) standard for its core automation model, but it uses the [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/) for Chromium and Playwright's own browser-specific transport layers for Firefox and WebKit.

Playwright controls browser navigation using methods like `page.goto(url)`. To interact with elements in the browser, tests must first locate them. Playwright offers different locator strategies:

- **CSS selectors**: Select elements using standard CSS syntax, e.g. `.btn-primary` or `input[name="email"]`. Fast and widely supported.
- **Text locators**: Select elements by their visible text content. Playwright provides `page.getByText()` for human-readable text queries.
- **Test IDs**: Select elements by a dedicated `data-testid` attribute, e.g. `page.getByTestId('submit-btn')`. This decouples tests from markup structure and styling.
- **ARIA roles**: Select elements by their semantic role and accessible name, e.g. `page.getByRole('button', { name: 'Submit' })`. This approach promotes accessibility-first development.
- **XPath**: A powerful but fragile and verbose querying language. Its use is strongly discouraged because it is tightly coupled to DOM structure and is hard to maintain.

The recommended approach is to prefer accessible locators (role, label, text) and `data-testid` attributes over structural selectors that leak implementation details.

### Auto-Waiting

One of the most common sources of flakiness in functional tests is poor synchronization between the test and the application state. In practice, the default strategy should be [auto-waiting](https://playwright.dev/docs/actionability). Actions such as `click`, `fill`, and `press`, and Playwright assertions such as `expect(locator).toBeVisible()`, already wait for element readiness and stable conditions. This is why fixed delays such as `waitForTimeout()` are a bad default because they slow down the suite, hide real timing problems, and still fail under variable load.

Explicit waiting is still necessary in specific scenarios, but it should be used intentionally and with clear responsibility. In other words, when auto-waiting is not enough, the test author is responsible for choosing a concrete and meaningful synchronization signal. Typical examples include waiting for navigation (`waitForURL`), waiting for a specific network response (`waitForResponse`), waiting for a loading indicator to disappear, or waiting for an asynchronous backend process to finish.

A practical rule is to use auto-waiting by default and add explicit waits only for domain-specific events that the framework cannot infer on its own. This keeps tests faster, clearer, and more reliable.

### Mocking APIs

Functional tests can intercept and mock network requests to control the data the application receives without depending on a live backend. In [Playwright's API Mocking](https://playwright.dev/docs/mock), `page.route()` can block a request completely and return fixture data with `route.fulfill()`, which is useful when the scenario needs full control over backend responses. For more details, see next example:

```typescript
describe('Promotions', () => {
  beforeEach(async ({ page }) => {
    await page.route('*/**/api/v1/promotions', async route => {
      const json = [{ name: 'Summer Sale', id: 21 }];

      await route.fulfill({ json });
    });
  });
});
```

When the test should keep most of the real response but adjust only a small part of it, `route.fetch()` can retrieve the original payload and `route.fulfill()` can return a patched version.

Alternatives such as [ng-apimock](https://www.npmjs.com/package/@ng-apimock/core) integrate the same idea into the Angular testing workflow and are often preferable in Angular-heavy projects because they centralize mocks outside the Playwright specs. This keeps functional tests focused on user behavior instead of polluting test code with inline API-mocking setup. API mocking still makes tests faster and more reliable, avoids polluting real databases with test data, and lets the team validate edge cases that may be difficult to reproduce on demand.

When the behavior under test depends not on HTTP but on browser-provided capabilities, Playwright also supports [mocking browser APIs](https://playwright.dev/docs/mock-browser-apis), typically by installing the mock before page load with `page.addInitScript()`.

### Mocking Clock

Time-dependent features such as inactivity logout, countdowns, throttling windows, and scheduled refreshes should use a mocked clock so tests stay deterministic and fast.

Playwright provides a dedicated [Clock API](https://playwright.dev/docs/clock). Utilizing `Clock` functionality allows developers to manipulate and control time within tests. The recommended approach is to use `setFixedTime` to set the time to a specific value. If that doesn't work for our use case, we can use `install` which allows us to pause time later on, fast forward it, tick it, etc. `setSystemTime` is only recommended for advanced use cases.

To avoid polluting test cases with repeated time setup, keep clock configuration in fixtures or `beforeEach` blocks and let the test body focus on user behavior and expected outcomes.

### Page Objects

[Page Objects](https://playwright.dev/docs/pom), also called the Page Object Model (POM), use an object-oriented structure to represent the UI. In this model, each page or major UI component has a corresponding class that centralizes locators and exposes reusable interaction methods. This model is especially useful in medium and large automation suites where many tests depend on the same screens and page states.

The main characteristics are:

- **Structure**: Object-oriented classes, usually one page object class per file;
- **Focus**: UI representation (elements, states, and page-specific actions);
- **Pros**: Centralizes locators, reduces duplication, and improves readability and maintainability;
- **Cons**: Can become bloated or over-engineered in small projects or in UIs that change very quickly.

The pattern usually follows some practical rules:

- Provide an interface that is easy to use from tests;
- Encapsulate locator and interaction details inside the page object;
- Model meaningful pages/components instead of creating classes for every tiny fragment;
- Return another page object when a method navigates to a different page;
- Keep assertions in test files, not in page object classes;
- Keep one page object class per file.

Since elements are encapsulated in these objects, UI changes are generally fixed in one place rather than across many tests.

In the example below, we can see a page object for promotional sections:

```typescript
export class PromotionalSectionPO {
  constructor(private readonly page: Page) { }

  getAllSections(): Locator {
    return this.page.locator('section');
  }

  getAllPromotions(section: Locator): Locator {
    return section.locator('promotion');
  }

  getPromotionName(promotion: Locator): Locator {
    return promotion.locator('.title');
  }
}
```

Each section contains a list of promotions. Each promotion contains its own details. When we need to execute some assertions for a specific promotion, instead of creating a new method, we can reuse a generic method that receives a promotion element.

### Helper Functions

Helper functions use a functional structure and focus on behavior and workflows rather than on page structure. They are useful for multi-step operations such as `loginUser`, `checkoutItem`, data setup, API preparation, and repeated domain actions.

The main characteristics are:

- **Structure**: Functional modules and composable functions;
- **Focus**: Reusable workflows and behavior-oriented operations;
- **Pros**: Fast to write for simple scenarios, reusable across suites, and no class hierarchy overhead;
- **Cons**: If helpers also own locators, UI changes may force updates in many functions unless locator definitions are centralized.

Unlike a Page Object, a Helper should not model a specific screen. Its responsibility is to orchestrate reusable operations that may span pages, API calls, fixtures, and test data.

### Flaky Tests

As the test suite grows, a small amount of instability can quickly turn the regression signal into noise. A flaky test is one that passes and fails without a code change, usually because it depends on timing, shared state, brittle selectors, an unstable environment, browser automation limits, or test-order coupling. Flaky tests are worse than no tests because they reduce trust in the suite and make it harder to tell whether a failure represents a real regression or only a temporary failure in the test itself.

The right response is to make the test deterministic, isolate the dependency that is introducing nondeterminism, or remove the unstable scenario from the lower-level suite until it can be stabilized. Temporarily disabling a flaky test can be a pragmatic short-term step, but it should be treated as technical debt rather than a permanent solution.

When a test still needs a safety net, retries can help separate transient failures from stable regressions. By default a test is not retried, but when retries are enabled, a test that passes on retry is marked as flaky rather than healthy, which makes the issue visible in the [HTML reporter](https://playwright.dev/docs/test-reporters#html-reporter). In practice, retries should help us gather evidence and reduce noise while we fix the root cause, not hide instability behind a passing build.

### Test Reporters

Test reporters for [Playwright Test](https://playwright.dev/docs/test-reporters) are responsible for turning raw execution results into feedback that different audiences can consume. In practice, teams combine lightweight console reporters such as [list](https://playwright.dev/docs/test-reporters#list-reporter), [line](https://playwright.dev/docs/test-reporters#line-reporter), and [dot](https://playwright.dev/docs/test-reporters#dot-reporter) for quick local runs with structured outputs such as [json](https://playwright.dev/docs/test-reporters#json-reporter), [junit](https://playwright.dev/docs/test-reporters#junit-reporter), and [blob](https://playwright.dev/docs/test-reporters#blob-reporter) for CI ingestion, trend analysis, and cross-shard merge workflows.

For human-readable diagnostics, the built-in [HTML reporter](https://playwright.dev/docs/test-reporters#html-reporter) is usually the central artifact because it links test status to retries, errors, traces, screenshots, videos, and timing in one place, which makes triage significantly faster in larger suites. Many teams extend this baseline with ecosystem reporters such as [allure-playwright](https://www.npmjs.com/package/allure-playwright) for richer historical dashboards and test metadata, or [playwright-html-reporter](https://www.npmjs.com/package/playwright-html-reporter) for alternative presentation and distribution needs, while keeping reporting strategy focused on signal quality, failure diagnosis speed, and long-term visibility of stability and flakiness trends.

### Debugging

End-to-end tests can be difficult to debug because they depend on an entire system, may depend on prior actions (such as log-in), and may change the state of the application they're testing.

For daily debugging, prefer IDE integrations over command-line-only workflows. In VS Code, the [Playwright Test for VSCode](https://marketplace.visualstudio.com/items?itemName=ms-playwright.playwright) extension and the Testing panel make it easier to run a single scenario, debug with breakpoints, inspect traces, and re-run only failed tests. This reduces context switching and speeds up investigation loops.

For deeper diagnosis, combine Playwright Inspector with browser developer tools to inspect network calls, console errors, DOM state, and timing behavior during each test step. In the [Debugging](https://playwright.dev/docs/debug) section of Playwright documentation, we have procedures to debug functional tests.

### Best Practices

The following practices focus on writing functional tests that are stable, maintainable, and aligned with real user behavior:

- [ ] **Have simple project structure**: A good project structure helps everyone catch up quickly, even on large-scale codebases. Keep an `e2e/` folder at the top level for end-to-end tests and test-specific configuration files. In multi-project workspaces, keep app-specific end-to-end tests in `projects/project-name/e2e/`.

- [ ] **Follow the Given-When-Then pattern**: The [Given-When-Then](https://martinfowler.com/bliki/GivenWhenThen.html) formula is a template intended to guide the writing of functional tests.

- [ ] **Make use of lint rules**: Linters can catch severe issues like incorrectly handled errors and lost information. Use plugins designed for test code patterns, such as [eslint-plugin-playwright](https://www.npmjs.com/package/eslint-plugin-playwright) for waiting and locator reliability.

- [ ] **Do not duplicate test coverage**: Do not create functional tests for areas already covered well by lower-level tests. If a UI component can be covered by a unit test, prefer that approach. Unit tests are easier to maintain, less flaky, and cheaper in CI, while functional tests should focus on big-picture user stories across many components and views.

- [ ] **Do not test everything**: Maximize coverage with fewer tests and avoid excessive granularity that creates maintenance burden. It is recommended to verify functional outcomes rather than internal implementation details.

- [ ] **Do not ignore flaky tests**: Tests depend on external systems and can fail randomly due to network conditions, system load, and other non-deterministic factors. Each flaky test needs targeted stabilization (better waits, setup fixes, etc.) or to be removed. Metrics such as execution time and failure/success trends can help identify root causes.

- [ ] **Do not add logic to the tests**: Avoid `if` statements and `loops` in test bodies. Extra logic can let tests pass without asserting real behavior and can slow execution. Tests should stay one-concern and straightforward.

- [ ] **Do not use XPath locator**: XPath is often the slowest and most brittle locator strategy, and it tends to break when markup changes. Expressions are hard to read and debug, and maintenance cost is high. Prefer alternative locator strategies whenever possible.

- [ ] **Do not use IDs**: IDs are often assumed stable but can change, and frameworks may generate or rely on them internally. Avoid adding IDs only for testing. Prefer user-facing locators and dedicated testing attributes like `data-testid`, and use IDs only when no better option exists.

- [ ] **Prefer user-facing and test-id locators**: In Playwright, use resilient locators such as `getByRole`, `getByLabel`, `getByPlaceholder`, and `getByTestId`. Avoid brittle selectors tied to implementation details, like deep CSS chains and dynamic class names. Use `getByText` only when text is intentional and stable.

- [ ] **Use page objects**: Page Object is a key design pattern for cleaner tests. It encapsulates page element knowledge, supports reuse across tests, and centralizes updates when UI templates change. This significantly improves maintainability for automation teams.

- [ ] **Test case only call page object's methods**: For reusability and maintainability, test cases should call page object methods instead of interacting with controls directly. Even simple clicks should be wrapped so requirement and UI changes require refactoring in one place only.

- [ ] **One file, one-page object class**: Define one page object class per file and use user-friendly names for files and classes. This structure improves project recognizability and makes page objects easier to locate in both small and large projects.

- [ ] **Do not place assertions inside page objects**: Keep page objects focused on interaction abstraction and avoid verification points inside them. Putting assertions in page object classes violates single responsibility and makes test scripts harder to understand and maintain.

- [ ] **Do not place assertions inside of try/catch blocks**: Assertions are meant to fail loudly and notify the test engine when expectations are not met. Wrapping assertions in try/catch can hide failures and reduce build feedback quality. Expect errors directly when an error is the intended outcome.

- [ ] **Make tests independent at the file level**: Playwright can run files in parallel across workers and browsers, and [Selenium Grid](https://www.selenium.dev/documentation/grid/) supports similar parallel cross-browser execution. File-level independence is critical because execution order is not guaranteed and isolated runs should remain reliable.

- [ ] **Navigate to the page under test before each test**: Start each test from a clean page state by navigating before execution. Also maintain route-navigation suites for major app paths to verify app connectivity, user navigation flow, and permission behaviors.

- [ ] **Use async & await**: Prefer `async` and `await` to avoid [Callback Hell](http://callbackhell.com/) and [Pyramid of Doom](https://en.wikipedia.org/wiki/Pyramid_of_doom_(programming)) style code. With Promise-based tools like Playwright and Puppeteer, async/await keeps test flows flatter, clearer, and easier to maintain.

- [ ] **Mock all API calls**: This is controversial, with different strategies across teams. Some prefer mocks for all tests to avoid network dependencies, but others prefer full-system realism. A practical compromise is maintaining smoke tests in production-like environments with real data.

- [ ] **Avoid hard waits**: Fixed delays like `waitForTimeout()` are fragile and create slow feedback loops. Increasing timeout values only adds latency and masks real synchronization issues. Prefer Playwright [auto-waiting](https://playwright.dev/docs/actionability) and [web-first assertions](https://playwright.dev/docs/api/class-playwrightassertions) like visibility and URL expectations.

## Visual Tests

### Fundamentals

[Visual testing](https://www.geeksforgeeks.org/software-testing/software-testing-visual-testing/) is necessary to make sure that style issues don’t pop up and the web application continues to look just as good as it works. Visual testing mostly focuses on visual aspects of the user interface. It allows us to pinpoint every pixel at the UI level so that HTML, CSS, and JavaScript differences don’t affect the user experience. Verification of visual aspects can be made manually or with the help of automation.

To make this practical, visual regression follows a simple cycle. First, it is important to create a baseline. It is the first screenshot capture of each step. Second, run another round of tests to check whether implemented changes break the application UI. When the test run is finished, we need to compare the latest test run with the baseline to verify whether any changes appeared. If so, we need to check whether the difference is something the development team intended to do. In other words, we need to check if changes appeared for the right reason. If it turns out that the development team did not intend to implement any of them, then they are called visual bugs. In the last phase, the baseline should be updated with new changes to validate new regression iterations properly. This cycle is represented in the next image.

![](data:image/png;base64,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)

Fig. 2: Visual regression

In practice, there are many tools and libraries that help testers and developers in building test scenarios, but all are focused on:

- Properly rendered and displayed graphics on the page;
- Components of a page displayed correctly, e.g. buttons with proper color and proper shape;
- Layout of a page e.g. each component is in right place on a page;
- Validation of component, if it does not overlay each other.

By comparison, in functional tests we take the following aspects of the application into consideration:

- Validation of input fields;
- Page navigation check;
- Business logic check, e.g. checking CRUD operation on a page;
- Check if hyperlinks are working.

Taken together, functional and visual tests provide the development team with a strong tool for verifying changes during the software development life cycle.

### Screenshot Tests

Tests are written to prevent mistakes from becoming user-facing issues. So far, we have focused only on testing application logic. However, the visual appearance of a web app is also important. One way to verify the app's appearance is to use a test that fails when a screenshot changes.

Automated tests that verify visual appearance can be hard to build. Still, we can create a simple screenshot test that fails when major, unintended visual changes appear. We can use [Playwright snapshots](https://playwright.dev/docs/test-snapshots) or [jest-image-snapshot](https://www.npmjs.com/package/jest-image-snapshot) to compare a browser screenshot against a reference image. However, comparing images is more complex than comparing text. Several factors can affect screenshot testing, such as the screen resolution on the machine running the tests. Fonts can also cause issues because they may render differently across machines and browsers.

Taking screenshots in Playwright is straightforward. Below, we have an example using Playwright for visual comparison. We can see that we are capturing a screenshot and comparing it to a baseline snapshot.

```typescript
describe('Promotional Sections', () => {
  beforeEach(async ({ page }) => {
    await page.goto('/promotions');
  });

  it('should compare successfully with a baseline', async ({ page }) => {
    await expect(page).toHaveScreenshot('example-page.png');
  });
});
```

In short, this test quickly tells us if the page starts looking different by mistake.

### Best Practices

Most best practices for functional testing also apply to visual testing. The following practices focus on creating visual tests that are stable, reviewable, and scalable over time:

- [ ] **Name the screenshots properly**: Initially, screenshots are given default names that are not too descriptive, so include a name so we know what we are actually looking at.

- [ ] **Do not try to test everything**: Try to maximize coverage with fewer tests. This is difficult because we want to know everything that is going on, but the more granular our tests are, the easier it is to track down the actual bugs causing the problem.

- [ ] **Avoid full-page screenshots**: Keep the tests modular or it will get annoying. Also, bigger screenshots mean longer processing time.

- [ ] **Freeze dynamic content**: Mock or control clocks, time zones, and locale-sensitive formatting so date/time labels do not change between runs. This prevents false positives caused by environment-dependent rendering.

- [ ] **Control viewport and browser settings**: Standardize viewport size, device scale factor, browser version, and color scheme in the visual test configuration. Snapshot consistency depends on a predictable rendering environment.

- [ ] **Stabilize fonts and assets**: Use packaged fonts and wait for font loading before taking screenshots. Missing fonts or late-loading assets can shift layout and cause misleading visual diffs.

- [ ] **Mask volatile UI areas**: Exclude or mask unstable UI areas such as advertisements, animated banners, rotating carousels, and user-specific avatars. Focus the comparison on areas that represent product quality.

- [ ] **Disable animations and transitions**: Turn off CSS animations and transitions during snapshot capture. Motion between frames can make deterministic comparison impossible even when the UI is correct.

- [ ] **Use component-level and page-level coverage**: Combine targeted component snapshots with a smaller set of page snapshots. This balances fast diagnosis with confidence in integrated layouts.

- [ ] **Define explicit diff thresholds and policy**: Set clear per-suite tolerance rules and review criteria for acceptable pixel differences. Teams should agree when to accept, reject, or rebaseline a change.

- [ ] **Review snapshots in pull requests**: Make visual diffs part of code review with clear ownership. Human approval of changed baselines helps prevent accidental acceptance of regressions.

- [ ] **Version and document baseline updates**: Record why baseline images changed and link them to intended UI changes. This creates traceability and reduces confusion during future regressions.

- [ ] **Plan for scaling visual testing**: If we are using git to store the baseline images and the project becomes increasingly larger, check out git storage tools like [Git Annex](https://git-annex.branchable.com/) or [Git Media](https://github.com/alebedev/git-media) to prevent the repo from growing out of control.

- [ ] **Use the pattern library**: If the team has a component library, map tests to individual components and their states. This makes it easier to localize failures and identify bugs by validating one component state at a time.

- [ ] **Run the tests frequently**: Build a continuous testing pipeline where the suite runs frequently, such as every day or on each repository change. The earlier issues are found, the lower the cost to fix them.

## Other Tests

### Accessibility Testing

Accessibility Testing is a type of testing which determines the usability of a product to the people having disabilities (deaf, blind, mentally disabled etc.). People with disabilities use assistive technology which helps them in operating a software product. Automated tools can quickly detect common issues such as missing labels, low color contrast, incorrect heading hierarchy, and invalid ARIA attributes.

### Cross Browser Testing

This testing ensures that software applications perform well on different hardware, operating systems, and network environments. The goal is to verify consistent behavior, not pixel-perfect similarity. Core user journeys should be executed in all supported browsers, while full regression can run in one primary browser plus a smaller compatibility suite in the others. Special attention is needed for CSS layout differences, date/time input behavior, file uploads, font rendering, and browser-specific security restrictions.

### Smoke Testing

Smoke testing is a lightweight verification step that checks whether the most critical parts of an application work after a new build or deployment. A smoke test usually confirms that the app starts correctly, the main routes load, essential APIs respond, and a few business-critical flows such as login or navigation are available. The goal is not to validate every detail, but to detect major failures quickly so that obviously broken builds do not move further in the delivery pipeline.

### Monkey Testing

Monkey testing is performed by a tester in a random manner, assuming that if the monkey uses the application, then how a monkey enters random values without any knowledge or understanding of the application. The most valuable outcome is not pass/fail status, but identifying unexpected crashes, frozen UI states, and unhandled exceptions that should be converted into deterministic regression tests. There are no special test cases written for monkey testing.

### Performance Testing

Performance testing evaluates the application's performance and ensures that it meets expected benchmark requirements. Performance must be measured, monitored, and refined continuously. In addition, the growing complexity of the web creates new challenges for tracking metrics, since results can vary significantly depending on device, browser, protocol, network type, and latency.

### Security Testing

Security testing for web applications should cover authentication, authorization, data protection, and dependency risks. Typical checks include role-based access control validation, token handling and expiration behavior, secure cookie settings, protection against common attacks such as XSS and CSRF, and safe handling of secrets in configuration files and logs. Automated dependency and container scans can complement security testing by identifying known vulnerabilities early in the delivery cycle.

## Gherkin

### Fundamentals

[Gherkin](https://cucumber.io/docs/gherkin/) is a domain-specific language, just like SQL and HTML. Each line is called a step, starts with a keyword, and ends with terminal punctuation. Tabs or spaces are used for indentation, and a comment can be added anywhere, but it should start with a # sign.

```gherkin
Feature: Guess the word

  # This scenario has three steps
  Scenario: Breaker joins a game
    Given the Maker has started a game with the word "silky"
    When the Breaker joins the Maker's game
    Then the Breaker must guess a word with 5 characters

```

Each non-blank line must start with a Gherkin keyword, followed by free-form text. The only exceptions are the feature and scenario descriptions.

Based on [Gherkin Reference](https://cucumber.io/docs/gherkin/reference/), the primary keywords are:

- **Feature**: The purpose of the *Feature* keyword is to provide a high-level description of a software feature, and to group related scenarios. The first primary keyword in a Gherkin document must always be *Feature*, followed by a “:” and a short text that describes the feature.
- **Rule**: The purpose of the Rule keyword is to represent one business rule that should be implemented. It provides additional information for a feature. A Rule is used to group together several scenarios that belong to this business rule.
- **Scenario (or Example)**: This is a concrete example that illustrates a business rule. It consists of a list of steps. Scenarios follow the pattern [Given-When-Then](https://martinfowler.com/bliki/GivenWhenThen.html).
- **Given, When, Then, And, But (steps)**: Steps are executed one at a time, in the sequence we’ve written them in. *Given* steps are used to describe the initial context of the system. *When* steps are used to describe an event, or an action. *Then* steps are used to describe an expected outcome, or result. *And* and *But* are used to avoid successive *Given’s*, *When’s*, or *Then’s*.
- **Background**: Occasionally we are repeating the same *Given* steps in all the scenarios in a *Feature*. A *Background* allows us to add some context to the scenarios that follow it. It can contain one or more *Given* steps, which are run before each scenario, but after any [*Before* hooks](https://cucumber.io/docs/cucumber/api/#hooks).
- **Scenario Outline (or Scenario Template)**: The *Scenario Outline* keyword can be used to run the same *Scenario* multiple times, with different combinations of values.
- **Examples**: A Scenario Outline must contain an *Examples* (or *Scenarios*) section(s). Its steps are interpreted as a template which is never directly run. Instead, the *Scenario Outline* is run once for each row in the *Examples* section beneath it (not counting the first header row).

There are a few secondary keywords as well:

```
  """ (Doc Strings)
  | (Data Tables)
  @ (Tags)
  # (Comments)
```

Functional tests can be written in Gherkin and executed with Cucumber + Playwright. Gherkin provides a business-readable format, Cucumber maps scenarios to test steps, and Playwright runs them in real browsers. Scenarios can include steps that sound very similar or even identical. It is recommended to look for opportunities to generalize steps and reuse them. The goal is to build a reusable library of steps so that adding new features requires less effort over time. Reusing steps is recommended whenever possible.

### Jira + Xray

[Jira](https://www.atlassian.com/software/jira) is widely used to manage user stories and delivery workflows, while [Xray](https://www.getxray.app/) extends Jira with full test management and traceability capabilities. Together, they can keep user stories, BDD scenarios, and execution evidence connected in one flow.

A practical approach to integrate Jira and Xray is:

1. Write or refine acceptance criteria in the Jira user story using business language;
2. Create an Xray Test issue (or use Xray's Cucumber/BDD Test type) linked to that story;
3. Add the Gherkin scenario directly in the Xray test so product owners, QAs, and developers can review it collaboratively;
4. Tag scenarios with issue keys (for example `@TEST-1234`) to preserve traceability between automated tests and Jira artifacts;
5. Execute the automated suite in [CI workflow](https://docs.getxray.app/space/XRAYCLOUD/44565056/Taking+advantage+of+Jira+Cloud+built-in+automation+capabilities) and import Cucumber JSON results back to Xray so test status, history, and evidence are visible in Jira dashboards and test plans.

This workflow improves requirement clarity, keeps BDD tests as living documentation, and provides end-to-end traceability from user story to automated execution.

### Cucumber + Playwright

[Cucumber](https://cucumber.io/) combined with Playwright (via [playwright-bdd](https://www.npmjs.com/package/playwright-bdd)) is a powerful combination for functional testing that bridges the gap between technical and non-technical team members. Cucumber allows test scenarios to be written in Gherkin, a plain-language format that non-technical stakeholders, business analysts, and product owners can read, understand, and validate. Playwright serves as the browser automation engine underneath, providing fast, reliable, and cross-browser testing capabilities.

Below is a practical example of how to configure and use Cucumber with Playwright:

**Step 1: Create a configuration file** (`playwright.config.js`):

```javascript
const testDir = defineBddConfig({
  features: 'features/**/*.feature',
  steps: 'steps/**/*.js',
});

export default defineConfig({
  testDir,
  reporter: 'html',
});
```

**Step 2: Create a feature file** (`features/login.feature`):

```gherkin
Feature: User Authentication

  Scenario: User can login with valid credentials
    Given I am on the login page
    When I enter email "user@example.com"
    And I enter password "password123"
    And I click the login button
    Then I should see the dashboard
    And URL should contain "dashboard"
```

**Step 3: Implement step definitions** (`steps/auth.js`):

```javascript
Given('I am on the login page', async ({ page }) => {
  await page.goto('https://app.example.com/login');
});

When('I enter email {string}', async ({ page }, email) => {
  await page.fill('input[type="email"]', email);
});

And('I enter password {string}', async ({ page }, password) => {
  await page.fill('input[type="password"]', password);
});

And('I click the login button', async ({ page }) => {
  await page.click('button:has-text("Login")');
});

Then('I should see the dashboard', async ({ page }) => {
  await expect(page).toHaveTitle(/Dashboard/);
});

And('URL should contain {string}', ({ page }, urlPart) => {
  expect(page.url()).toContain(urlPart);
});
```

**Step 4: Run the tests**:

To execute the functional tests, use the following npm script:

```json
{
  "scripts": {
    "e2e": "bddgen -c playwright.config.ts && playwright test -c playwright.config.ts"
  }
}
```

This setup provides several benefits, namely, feature files serve as living documentation, non-technical team members can contribute to test scenarios, Playwright's debugging and reporting tools are fully available, and the test results are automatically generated and can be viewed in the HTML report.

### AI + BDD

In the AI era, teams can accelerate Behavior-Driven Development without sacrificing quality by treating Gherkin as the shared language for both acceptance criteria and executable tests. A practical approach is to capture user-story acceptance criteria directly in Gherkin from day one, then evolve those same scenarios into automated tests with [Cucumber](https://cucumber.io/) and [Playwright](https://playwright.dev/). This keeps requirements, implementation, and validation aligned in a single artifact.

A practical flow is:

1. **Capture acceptance criteria in Gherkin**: Write acceptance criteria as executable Gherkin scenarios, so they become living documentation that product owners, QAs, and developers can review and understand together.
2. **Apply Gherkin-first TDD**: Write the scenario first (or reuse the acceptance criteria), let it fail, implement only what is needed to make it pass, then refactor without breaking it.
3. **Automate with Cucumber + Playwright**: Map approved scenarios to reusable step definitions, then run them in real browsers with Playwright. Keep steps declarative and reuse existing definitions whenever possible to avoid duplication.

This approach keeps feature files readable, improves cross-functional collaboration, and avoids unnecessary growth of duplicated test code. AI can further improve testing quality by drafting scenarios from requirements, suggesting edge and negative cases, highlighting ambiguity in acceptance criteria, generating realistic test data variations, and identifying opportunities to consolidate repetitive steps.

When using AI assistance in automation, [Playwright MCP](https://playwright.dev/docs/getting-started-mcp) can accelerate browser-test development by combining language-guided generation with real browser execution through Playwright. In practice, this improves selector quality, synchronization decisions, flaky-test triage, and debugging loops.

### Best Practices

The following best practices help teams create Gherkin scenarios that remain clear, maintainable, and reusable over time:

- [ ] **Define the actor of the system**: Be explicit about who the user is instead of mixing first-person and third-person language. Clear actor naming makes multi-user behaviour much easier to understand.

- [ ] **Use native language to write scenarios**: Gherkin supports multiple languages, so choose the one that best matches the team and product context. Readability matters more than forcing English when another language is clearer.

- [ ] **Write declarative scenarios**: Focus on behaviour and business value, not on clicks, keystrokes, or other implementation details. Avoid situations where each step is a precise instruction. Declarative steps make scenarios easier to read and less brittle when the UI or flow changes.

- [ ] **Use meaningful scenario names**: Give scenarios names that summarize the main idea in one line. Clear titles make reports easier to scan and failures easier to understand. Try to summarize the gist of a scenario in a single line.

- [ ] **Add good scenario descriptions**: Use descriptions when the title is not enough to explain the context or intent. A short explanation below the feature or scenario helps future readers understand why the scenario exists.

- [ ] **Write the scenario before writing the code**: Drafting scenarios first encourages product, QA, and development to align on behaviour before implementation starts. This reduces ambiguity and surfaces disagreements earlier.

- [ ] **Do not write scenarios in isolation**: Include the whole team when creating scenarios so the result stays automatable and reflects the real domain. Collaboration helps remove unnecessary detail and improves clarity.

- [ ] **Make scenarios independent and deterministic**: Each scenario should run the same way every time, regardless of execution order. Avoid hidden state, shared setup, or data that leaks across scenarios.

- [ ] **Remove unneeded scenarios**: Remove trivial scenarios once the behaviour is already covered by more valuable tests. Keeping them around often adds noise without helping the team understand the product.

- [ ] **Use backgrounds to reduce the number of steps**: Move repeated setup into a `Background` when several scenarios start the same way. This keeps the scenarios shorter and makes the common precondition obvious.

- [ ] **Avoid overuse of scenario outline**: `Scenario Outline` are useful for data-driven cases, but they can quickly multiply test count and execution time. Use them carefully, especially for slow UI-driven flows.

- [ ] **Limit the number of scenarios per feature**: Split large feature files when they start to feel crowded or hard to scan. Smaller files are easier to navigate and keep related cases grouped by purpose.

- [ ] **Limit the number of steps per scenario**: Keep scenarios focused and avoid long procedural chains. If a flow becomes too long, look for a more declarative step, a table, or a better feature split. Consider decreasing their number to some reasonable value, say, 2-3 "Ands" for one `Given`, `When`, or `Then` step.

- [ ] **Avoid using conjunctive steps**: Prefer one action per step when possible, because it keeps steps modular and reusable. Conjunctive steps can be acceptable, but they should be the exception rather than the default. The next bad example shows two actions connected with the keyword `and`:

  ```gherkin
  Given I am on the login page
  When I sign in with valid credentials
  Then I see the "Welcome User" message and the logout button
  ```

- [ ] **Use And and But keywords**: Use `And` and `But` to keep the structure of a scenario readable without repeating `Given`, `When`, or `Then`. This makes the flow easier to follow while preserving the intent. See the following example:

  ```gherkin
  Given I am on the login page
  When I sign in with valid credentials
  Then I see the "Welcome User" message
  And I see the logout button
  But I not see "You are not logged in" message
  ```

- [ ] **Refactor and reuse step definitions**: Scenarios can include steps that sound very similar or even the same. Reuse common step definitions when they express the same domain concept. A shared step library keeps the suite smaller and makes new scenarios cheaper to add.

- [ ] **Use tags to organize features and scenarios**: Tag scenarios by purpose, level, or execution frequency so they can be filtered and grouped cleanly. Tags make it easier to run smoke, integration, or other subsets on demand.

- [ ] **Use lint rules to check Gherkin syntax**: Linting helps keep scenario structure consistent across authors and catches style problems early. It is especially useful when multiple people contribute to the same suite. Use [Gherkin lint](https://www.npmjs.com/package/gherkin-lint) to force a consistent style.

- [ ] **Avoid testing through the UI**: Prefer business-facing scenarios over button-click scripts, because UI details are slower and more brittle. Gherkin should describe intent, not merely mirror the interface.

- [ ] **Features are not user stories**: A feature file may collect several related stories, so do not force a one-to-one mapping. Keep the file focused on the behaviour the team needs to understand, then add scenarios only when they add value.

## Testing Suites

Running unit tests should be fast to keep the feedback cycle short, especially in TDD-style development. In Angular projects, component template rendering is often the main execution bottleneck. For this reason, it's recommended to use the *.i.spec.ts* postfix for integration tests and keep *.spec.ts* for unit tests. For e2e tests, it's recommended to split tests into functional and visual suites.

To make execution simple and consistent, expose each suite through npm scripts. This allows teams to run *npm run test*, *npm run test:unit*, *npm run test:integration*, *npm run e2e*, *npm run functional*, and *npm run visual*, with additional watch-mode variations. For more details, see next npm scripts:

```json
{
  "scripts": {
    "ng": "ng",
    "start": "ng serve",
    "test": "npm-run-all -s test:unit test:integration",
    "test:unit": "ng test -c unit --code-coverage",
    "test:unit:watch": "ng test -c unit --code-coverage --watch",
    "test:integration": "ng test -c integration --code-coverage",
    "test:integration:watch": "ng test -c integration --code-coverage --watch",
    "e2e": "playwright test -c e2e/playwright.config.ts",
    "functional": "playwright test -c e2e/playwright.config.ts --grep @functional",
    "functional:watch": "npm-run-all -p start playwright:watch",
    "visual": "playwright test -c e2e/playwright.config.ts --grep @visual",
    "visual:watch": "npm-run-all -p start playwright:watch",
    "playwright": "playwright test -c e2e/playwright.watch.config.ts",
    "playwright:watch": "watch \"npm run playwright\" e2e/test --wait=1"
  }
}
```

In Angular’s *angular.json*, we can define separate test configurations through the *test.configurations* property for all tests, unit tests, and integration tests, including watch-mode variations. We can follow the same structure for e2e tests, also with watch-mode variations. The *npm run e2e* script runs all suites, while *npm run functional* and *npm run visual* target functional and visual suites, respectively.

For interactive watch workflows, auxiliary scripts can call testing frameworks using [npm-run-all](https://www.npmjs.com/package/npm-run-all) and [watch](https://www.npmjs.com/package/watch).

## Final Remarks

In this chapter, we covered the main testing layers for Angular applications: isolated and shallow unit tests, integration tests, and end-to-end testing through functional and visual suites.

We looked at a pragmatic strategy that balances confidence, speed, and maintenance cost, with practical guidance on test design, tooling (Vitest/Jest, Playwright, and Cucumber), and when each test type adds the highest value. We also saw how to keep feedback loops fast by separating unit and integration suites with naming conventions, organizing npm scripts for targeted execution, and identifying slow tests that should be promoted to higher-level suites.

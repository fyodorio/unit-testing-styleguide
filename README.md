# Front End Unit Testing Style Guide 
#### BDD with Jest/Jasmine/Mocha for TypeScript/Angular

## Purpose

The purpose of this style guide is to offer suggested best practices when writing Front End unit tests using Jest, Jasmine or Mocha. Though this Style Guide is written specifically for using with TypeScript and Angular, most of its concepts can be applied to testing under a wide range of different JavaScript technologies and code bases.

## Introduction

Jest is the spec-style unit testing library following Behaviour Driven Development (BDD) principles. It's API is almost fully compatible with Jasmine and Mocha API, so the basic principles described here can be used for all the three mentioned unit testing frameworks. This document governs three main parts of the unit testing process for the needs of Front End Development:

* [Code preparation](#best-practices-for-providing-code-testability)
* [Evaluating the scope of testing](#requirements-for-test-coverage)
* [Scaffolding the environment and tools](#environment-related-requirements)
* [Writing unit tests](#rules-for-writing-the-tests)

## Best practices for providing code testability

1. All the code architecture should follow the Single Responsibility Principle to achieve Separation of Concerns inside of it 
1. API methods should not be tightly coupled with the specific implementation (data sources)
1. Pure functions should be preferred when writing the API methods
1. Methods should be decomposed as much as possible
1. Business logic should be delegated from components to services as much as possible
1. All the API methods and properties should use types/enums/interfaces (including method attributes and return values)
1. Code inside a file should be documented ("as much as needed but not more" (c) Albert Einstein) prior to be tested

## Requirements for test coverage

1. Priority chain for code base test coverage: CORE -> SHARED -> MODULES
1. All the components and services should be covered with smoke tests (at least)
1. Service API should be covered with tests
1. Only the publicly-exposed API should be tested 

## Environment-related requirements

1. Each `*.spec.ts` file with specific test suite in it should be placed alongside the component/service/etc being tested and named accordingly
1. Uppermost `describe` block of the test suite should be named after component/service/etc being tested to find the source of fail faster after test runs
1. Test should be run in isolation - all the dependencies should be doubled (mocked/stubbed)
1. Avoid mocks (focusing on internals) in favor of stubs and spies (focusing on requirements/specific functionality)
1. Minimize external helpers and abstractions
1. Avoid global test fixtures and seeds, add data per-test
1. Use Angular utilities (from `@angular/core/testing`) to pre-compile the Angular-specific code (e.g. `TestBed` and `ComponentFixture` for scaffolding, `async` for make compilation asynchronous and wait for it) and get access to internals of the pre-compiled test fixtures (e.g. `detectChanges` which isn't run automatically, or `debugElement` which allows to access DOM and produce events)

## Rules for writing the tests

1. [Testing Is Easy](#testing-is-easy)
1. [Speak Human](#speak-human)
1. [Write _Unit_ Tests](#write-unit-tests)
1. [Arrange-Act-Assert](#arrange-act-assert)
1. [Don't Repeat Yourself](#dont-repeat-yourself)
1. [`this` Is How We Do It](#this-is-how-we-do-it)
1. [Avoid the `All`s](#avoid-the-alls)
1. [Be `describe`tive](#be-describetive)
1. [Write Minimum Passable Tests](#write-minimum-passable-tests)
1. [Randomize input data](#randomize-input-data)

### Testing Is Easy

Don't be afraid to write unit test, and don't overthink the process: 

1. `describe` what your testing. This is your test `suite`.
1. `it` should have some expected behaviors. These are your `specs`.
1. `expect` or assert these behaviors to hold true. These are your `expectations`

```typescript
describe('MyAwesomeComponent', () => {
  beforeEach( () => {
    // reproduce the test state
  })

  it('should be awesome', () => {
    expect(component).toBe(awesome)
  });

  // More specs here

})
````

### Speak Human

Label your test suites (`describe` blocks) and specs (`it` blocks) in a way that clearly conveys the intention of each unit test. Note that the name of each test is the title of its `it` preceded by all its parent `describe` names. Favor assertive verbs and avoid ones like "should."

#### Why?

* Test suite becomes documentation for your codebase (helpful for new team members and non-technical stakeholders)
* Failure messages accurately depict what is broken
* Forces good naming conventions in tested code

#### Bad:

```typescript
// Output: "Array adds to the end"
describe('Array', () => {
  it('adds to the end', () => {
    const initialArray = [1];
    initialArray.push(2);
    expect(initialArray).toEqual([1, 2]);
  });
});
```

#### Better:

```typescript
// Output: "Array.prototype .push(x) appends x to the end of the Array"
describe('Array.prototype', () => {
  describe('.push(x)', () => {
    it('appends x to the end of the Array', () => {
      const initialArray = [1];
      initialArray.push(2);
      expect(initialArray).toEqual([1, 2]);
    });
  });
});
```

### Write _Unit_ Tests

A unit test should test **one** thing. Confine your `it` blocks to a single assertion.

#### Why?

* Single responsibility principle
* A test can fail for only one reason

#### Bad:

```typescript
describe('Array.prototype', () => {
  describe('.push(x)', () => {
    it('appends x to the end of the Array and returns it', () => {
      const initialArray = [1];
      expect(initialArray.push(2)).toBe(2);
      expect(initialArray).toEqual([1, 2]);
    });
  });
});
```

#### Better:

```typescript
describe('Array.prototype', () => {
  describe('.push(x)', () => {
    it('appends x to the end of the Array', () => {
      const initialArray = [1];
      initialArray.push(2);
      expect(initialArray).toEqual([1, 2]);
    });

    it('returns x', () => {
      const initialArray = [1];
      expect(initialArray.push(2)).toBe(2);
    });
  });
});
```

### Arrange-Act-Assert

Organize your code in a way that clearly conveys the 3 A's of each unit test. One way to accomplish this is by Arranging and Acting in `before` blocks and Asserting in `it` ones.

#### Why?

* The AAA unit test pattern is well known and recommended
* Improves unit test modularity and creates opportunities to DRY things up

#### Bad:

```typescript
describe('Array.prototype', () => {
  describe('.push(x)', () => {
    it('appends x to the end of the Array', () => {
      const initialArray = [1];
      initialArray.push(2);
      expect(initialArray).toEqual([1, 2]);
    });
  });
});
```

#### Better:

```typescript
describe('Array.prototype', () => {
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

### Don't Repeat Yourself

Use `before`/`after` blocks to DRY up repeated setup, teardown, and action code.

#### Why?

* Keeps test suite more concise and readable
* Changes only need to be made in one place
* Unit tests are not exempt from coding best practices

#### Bad:

```typescript
describe('Array.prototype', () => {
  describe('.push(x)', () => {
    it('appends x to the end of the Array', () => {
      const initialArray = [1];
      initialArray.push(2);
      expect(initialArray).toEqual([1, 2]);
    });

    it('returns x', () => {
      const initialArray = [1];
      expect(initialArray.push(2)).toBe(2);
    });
  });
});
```

#### Better:

```typescript
describe('Array.prototype', () => {
  describe('.push(x)', () => {
    let initialArray,
        pushResult;

    beforeEach(() => {
      initialArray = [1];

      pushResult = initialArray.push(2);
    });

    it('appends x to the end of the Array', () => {
      expect(initialArray).toEqual([1, 2]);
    });

    it('returns x', () => {
      expect(pushResult).toBe(2);
    });
  });
});
```

### `this` Is How We Do It

Use `this` to share variables between `it` and `before`/`after` blocks.

#### Why?

* Declare and initialize variables on one line
* Most testing frameworks automatically clean the `this` object between specs to avoid state leak

#### Bad:

```typescript
describe('Array.prototype', () => {
  describe('.push(x)', () => {
    let initialArray,
        pushResult;

    beforeEach(() => {
      initialArray = [1];

      pushResult = initialArray.push(2);
    });

    it('appends x to the end of the Array', () => {
      expect(initialArray).toEqual([1, 2]);
    });

    it('returns x', () => {
      expect(pushResult).toBe(2);
    });
  });
});
```

#### Better:

```typescript
describe('Array.prototype', () => {
  describe('.push(x)', () => {
    beforeEach(() => {
      this.initialArray = [1];

      this.pushResult = this.initialArray.push(2);
    });

    it('appends x to the end of the Array', () => {
      expect(this.initialArray).toEqual([1, 2]);
    });

    it('returns x', () => {
      expect(this.pushResult).toBe(2);
    });
  });
});
```

### Avoid the `All`s

Prefer `beforeEach/afterEach` blocks over `beforeAll/afterAll` ones. The latter are not reset between tests.

#### Why?

* Avoids accidental state leak
* Enforces test independence
* Order of `All` block execution relative to `Each` ones is not always obvious

#### Bad:

```typescript
describe('Array.prototype', () => {
  describe('.push(x)', () => {
    beforeAll(() => {
      this.initialArray = [1];
    });

    beforeEach(() => {
      this.pushResult = this.initialArray.push(2);
    });

    it('appends x to the end of the Array', () => {
      expect(this.initialArray).toEqual([1, 2]);
    });

    it('returns x', () => {
      expect(this.pushResult).toBe(2);
    });
  });
});
```

#### Better:

```typescript
describe('Array.prototype', () => {
  describe('.push(x)', () => {
    beforeEach(() => {
      this.initialArray = [1];

      this.pushResult = this.initialArray.push(2);
    });

    it('appends x to the end of the Array', () => {
      expect(this.initialArray).toEqual([1, 2]);
    });

    it('returns x', () => {
      expect(this.pushResult).toBe(2);
    });
  });
});
```

### Be `describe`tive

Nest `describe` blocks liberally to create functional subsets.

#### Why?

* Allows tests to build on each other from least to most specific
* Creates tests that are easy to extend and/or refactor
* Makes branch testing easier and less repetitive
* Encapsulates tests based on their common denominator

#### Bad:

```typescript
describe('Array.prototype', () => {
  describe('.push(x) on an empty Array', () => {
    beforeEach(() => {
      this.initialArray = [];

      this.initialArray.push(1);
    });

    it('appends x to the Array', () => {
      expect(this.initialArray).toEqual([1]);
    });
  });

  describe('.push(x) on a non-empty Array', () => {
    beforeEach(() => {
      this.initialArray = [1];

      this.initialArray.push(2);
    });

    it('appends x to the end of the Array', () => {
      expect(this.initialArray).toEqual([1, 2]);
    });
  });
});
```

#### Better:

```typescript
describe('Array.prototype', () => {
  describe('.push(x)', () => {
    describe('on an empty Array', () => {
      beforeEach(() => {
        this.initialArray = [];

        this.initialArray.push(1);
      });

      it('appends x to the Array', () => {
        expect(this.initialArray).toEqual([1]);
      });
    });

    describe('on a non-empty Array', () => {
      beforeEach(() => {
        this.initialArray = [1];

        this.initialArray.push(2);
      });

      it('appends x to the end of the Array', () => {
        expect(this.initialArray).toEqual([1, 2]);
      });
    });
  });
});
```

### Write Minimum Passable Tests

If appropriate, use test framework's built-in matchers (such as `.toContain`, `.any`, `.stringMatching`, etc) to compare arguments and results. You can also create your own custom matcher functions.

#### Why?

* Tests become more resilient to future changes in the codebase
* Closer to testing behavior over implementation

#### Bad:

```typescript
describe('Array.prototype', () => {
  describe('.push(x)', () => {
    beforeEach(() => {
      this.initialArray = [];

      this.initialArray.push(1);
    });

    it('appends x to the Array', () => {
      expect(this.initialArray).toEqual([1]);
    });
  });
});
```

#### Better:

```typescript
describe('Array.prototype', () => {
  describe('.push(x)', () => {
    beforeEach(() => {
      this.initialArray = [];

      this.initialArray.push(1);
    });

    it('appends x to the Array', () => {
      expect(this.initialArray).toContain(1);
    });
  });
});
```

### Randomize input data

Avoid using `"foo"`, use realistic complex input data instead (`"$%JAFADF1313**@"`)

#### Why?

* Data complexity related bugs are caught earlier this way

## Contributing

This style guide ultimately represents the opinions of its contributors. If you disagree with anything, or wish to add more, please create an issue or submit a pull request. Our goal is to continuously improve the guide and build consensus around it.

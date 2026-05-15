# Testing Patterns Reference

스택 전반의 흔한 테스트 패턴에 대한 빠른 참조. `test-driven-development` skill과 함께 사용하세요.

## 목차

- [테스트 구조 (Arrange-Act-Assert)](#test-structure-arrange-act-assert)
- [테스트 명명 규칙](#test-naming-conventions)
- [흔한 Assertion](#common-assertions)
- [Mocking 패턴](#mocking-patterns)
- [React/컴포넌트 테스트](#reactcomponent-testing)
- [API / Integration 테스트](#api--integration-testing)
- [E2E 테스트 (Playwright)](#e2e-testing-playwright)
- [테스트 안티패턴](#test-anti-patterns)

## 테스트 구조 (Arrange-Act-Assert)

```typescript
it('describes expected behavior', () => {
  // Arrange: 테스트 데이터와 사전조건 설정
  const input = { title: 'Test Task', priority: 'high' };

  // Act: 테스트 대상 액션 수행
  const result = createTask(input);

  // Assert: 결과 검증
  expect(result.title).toBe('Test Task');
  expect(result.priority).toBe('high');
  expect(result.status).toBe('pending');
});
```

## 테스트 명명 규칙

```typescript
// 패턴: [unit] [expected behavior] [condition]
describe('TaskService.createTask', () => {
  it('creates a task with default pending status', () => {});
  it('throws ValidationError when title is empty', () => {});
  it('trims whitespace from title', () => {});
  it('generates a unique ID for each task', () => {});
});
```

## 흔한 Assertion

```typescript
// 동등성
expect(result).toBe(expected);           // 엄격한 동등성 (===)
expect(result).toEqual(expected);        // 깊은 동등성 (객체/배열)
expect(result).toStrictEqual(expected);  // 깊은 동등성 + 타입 매칭

// Truthiness
expect(result).toBeTruthy();
expect(result).toBeFalsy();
expect(result).toBeNull();
expect(result).toBeDefined();
expect(result).toBeUndefined();

// 숫자
expect(result).toBeGreaterThan(5);
expect(result).toBeLessThanOrEqual(10);
expect(result).toBeCloseTo(0.3, 5);      // 부동소수점

// 문자열
expect(result).toMatch(/pattern/);
expect(result).toContain('substring');

// 배열 / 객체
expect(array).toContain(item);
expect(array).toHaveLength(3);
expect(object).toHaveProperty('key', 'value');

// 에러
expect(() => fn()).toThrow();
expect(() => fn()).toThrow(ValidationError);
expect(() => fn()).toThrow('specific message');

// Async
await expect(asyncFn()).resolves.toBe(value);
await expect(asyncFn()).rejects.toThrow(Error);
```

## Mocking 패턴

### Mock 함수

```typescript
const mockFn = jest.fn();
mockFn.mockReturnValue(42);
mockFn.mockResolvedValue({ data: 'test' });
mockFn.mockImplementation((x) => x * 2);

expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');
expect(mockFn).toHaveBeenCalledTimes(3);
```

### Mock 모듈

```typescript
// 모듈 전체를 mock
jest.mock('./database', () => ({
  query: jest.fn().mockResolvedValue([{ id: 1, title: 'Test' }]),
}));

// 특정 export를 mock
jest.mock('./utils', () => ({
  ...jest.requireActual('./utils'),
  generateId: jest.fn().mockReturnValue('test-id'),
}));
```

### 경계에서만 Mock

```
이것들을 mock:                 이것들은 mock 안 함:
├── 데이터베이스 호출          ├── 내부 유틸리티 함수
├── HTTP 요청                  ├── 비즈니스 로직
├── 파일 시스템 작업           ├── 데이터 변환
├── 외부 API 호출              ├── 검증 함수
└── 시간/날짜 (필요 시)        └── 순수 함수
```

## React/컴포넌트 테스트

```tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';

describe('TaskForm', () => {
  it('submits the form with entered data', async () => {
    const onSubmit = jest.fn();
    render(<TaskForm onSubmit={onSubmit} />);

    // test ID가 아니라 접근 가능한 role/label로 요소 찾기
    await screen.findByRole('textbox', { name: /title/i });
    fireEvent.change(screen.getByRole('textbox', { name: /title/i }), {
      target: { value: 'New Task' },
    });
    fireEvent.click(screen.getByRole('button', { name: /create/i }));

    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({ title: 'New Task' });
    });
  });

  it('shows validation error for empty title', async () => {
    render(<TaskForm onSubmit={jest.fn()} />);

    fireEvent.click(screen.getByRole('button', { name: /create/i }));

    expect(await screen.findByText(/title is required/i)).toBeInTheDocument();
  });
});
```

## API / Integration 테스트

```typescript
import request from 'supertest';
import { app } from '../src/app';

describe('POST /api/tasks', () => {
  it('creates a task and returns 201', async () => {
    const response = await request(app)
      .post('/api/tasks')
      .send({ title: 'Test Task' })
      .set('Authorization', `Bearer ${testToken}`)
      .expect(201);

    expect(response.body).toMatchObject({
      id: expect.any(String),
      title: 'Test Task',
      status: 'pending',
    });
  });

  it('returns 422 for invalid input', async () => {
    const response = await request(app)
      .post('/api/tasks')
      .send({ title: '' })
      .set('Authorization', `Bearer ${testToken}`)
      .expect(422);

    expect(response.body.error.code).toBe('VALIDATION_ERROR');
  });

  it('returns 401 without authentication', async () => {
    await request(app)
      .post('/api/tasks')
      .send({ title: 'Test' })
      .expect(401);
  });
});
```

## E2E 테스트 (Playwright)

```typescript
import { test, expect } from '@playwright/test';

test('user can create and complete a task', async ({ page }) => {
  // 이동 및 인증
  await page.goto('/');
  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="password"]', 'testpass123');
  await page.click('button:has-text("Log in")');

  // task 생성
  await page.click('button:has-text("New Task")');
  await page.fill('[name="title"]', 'Buy groceries');
  await page.click('button:has-text("Create")');

  // task가 나타나는지 검증
  await expect(page.locator('text=Buy groceries')).toBeVisible();

  // task 완료
  await page.click('[aria-label="Complete Buy groceries"]');
  await expect(page.locator('text=Buy groceries')).toHaveCSS(
    'text-decoration-line', 'line-through'
  );
});
```

## 테스트 안티패턴

| 안티패턴 | 문제 | 더 나은 접근 |
|---|---|---|
| 구현 세부사항 테스트 | refactor 시 깨짐 | 입력/출력 테스트 |
| 모든 것을 snapshot | 아무도 snapshot diff를 리뷰 안 함 | 구체적인 값 assert |
| 공유 가변 상태 | 테스트가 서로 오염 | 테스트별 setup/teardown |
| 서드파티 코드 테스트 | 시간 낭비, 당신의 버그 아님 | 경계를 mock |
| CI 통과를 위해 테스트 skip | 실제 버그 숨김 | 테스트 수정 또는 삭제 |
| `test.skip`을 영구 사용 | 죽은 코드 | 제거 또는 수정 |
| 지나치게 광범위한 assertion | 회귀를 못 잡음 | 구체적으로 |
| async 에러 처리 없음 | 삼켜진 에러, 거짓 통과 | async 테스트는 항상 `await` |

# Implementation Plan: Whistleblower Earnings Dashboard

## Overview

This implementation adds a REST API endpoint that provides whistleblowers with visibility into their earnings history and payout status. The implementation follows existing backend patterns using Express.js routing, Zod validation, and the established error handling framework.

## Tasks

- [ ] 1. Set up Zod schemas for validation
  - [x] 1.1 Create whistleblower.ts schema file with request/response validation
    - Define whistleblowerIdParamSchema for path parameter validation
    - Define earningsTotalsSchema for totals response structure
    - Define earningsHistoryItemSchema for individual reward items
    - Define earningsResponseSchema for complete API response
    - Export TypeScript types using z.infer
    - _Requirements: 1.2, 1.3, 2.1, 3.1, 3.2, 3.4_
  - [ ]\* 1.2 Write unit tests for schema validation
    - Test valid schema inputs pass validation
    - Test invalid inputs fail with appropriate errors
    - Test optional fields (paidAt, USDC amounts)
    - _Requirements: 1.2, 2.1, 3.2_

- [ ] 2. Implement earnings service layer
  - [x] 2.1 Create earnings.ts service with aggregation logic
    - Define EarningsService interface with getEarnings method
    - Implement currency conversion from USDC to NGN
    - Implement aggregation logic for totals (total, pending, paid)
    - Implement history formatting and sorting (descending by createdAt)
    - Handle data layer queries and error cases
    - _Requirements: 1.2, 2.2, 2.3, 2.4, 2.5, 3.5_
  - [ ]\* 2.2 Write property test for total aggregation
    - **Property 2: Total Aggregation Correctness**
    - **Validates: Requirements 2.2, 2.5**
    - Verify totalNgn equals sum of all amountNgn values
    - Verify totalUsdc equals sum of all amountUsdc values
    - _Requirements: 2.2, 2.5_
  - [ ]\* 2.3 Write property test for pending aggregation
    - **Property 3: Pending Aggregation Correctness**
    - **Validates: Requirements 2.3, 2.5**
    - Verify pendingNgn equals sum of amountNgn for pending/payable status
    - Verify pendingUsdc equals sum of amountUsdc for pending/payable status
    - _Requirements: 2.3, 2.5_
  - [ ]\* 2.4 Write property test for paid aggregation
    - **Property 4: Paid Aggregation Correctness**
    - **Validates: Requirements 2.4, 2.5**
    - Verify paidNgn equals sum of amountNgn for paid status
    - Verify paidUsdc equals sum of amountUsdc for paid status
    - _Requirements: 2.4, 2.5_
  - [ ]\* 2.5 Write property test for aggregation partition
    - **Property 8: Aggregation Partition**
    - **Validates: Requirements 2.2, 2.3, 2.4, 2.5**
    - Verify pendingNgn + paidNgn equals totalNgn
    - Verify pendingUsdc + paidUsdc equals totalUsdc
    - _Requirements: 2.2, 2.3, 2.4, 2.5_
  - [ ]\* 2.6 Write unit tests for service layer
    - Test with empty reward set (zero totals, empty history)
    - Test with single reward
    - Test with mixed status rewards
    - Test currency conversion accuracy
    - Test error handling for data layer failures
    - _Requirements: 1.2, 2.2, 2.3, 2.4, 3.5_

- [x] 3. Checkpoint - Ensure service layer tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 4. Implement Express router
  - [x] 4.1 Create whistleblower.ts router with GET endpoint
    - Create factory function createWhistleblowerRouter accepting EarningsService
    - Define GET /api/whistleblower/:id/earnings route
    - Add Zod validation middleware for path parameters
    - Call earnings service and return JSON response
    - Handle errors using AppError pattern (404 for not found, 400 for validation)
    - _Requirements: 1.1, 1.2, 1.4_
  - [ ]\* 4.2 Write property test for response structure
    - **Property 1: Response Structure Completeness**
    - **Validates: Requirements 1.3, 2.1, 3.1**
    - Verify response contains totals object with required fields
    - Verify response contains history array
    - _Requirements: 1.3, 2.1, 3.1_
  - [ ]\* 4.3 Write property test for history item schema
    - **Property 5: History Item Schema Validity**
    - **Validates: Requirements 3.2, 3.3, 3.4**
    - Verify each history item has required fields
    - Verify status is one of pending/payable/paid
    - Verify paidAt present when status is paid
    - _Requirements: 3.2, 3.3, 3.4_
  - [ ]\* 4.4 Write property test for history ordering
    - **Property 6: History Ordering**
    - **Validates: Requirements 3.5**
    - Verify history items ordered by createdAt descending
    - _Requirements: 3.5_
  - [ ]\* 4.5 Write property test for error handling
    - **Property 7: Invalid ID Error Handling**
    - **Validates: Requirements 1.4**
    - Verify invalid/non-existent ID returns 404
    - _Requirements: 1.4_
  - [ ]\* 4.6 Write unit tests for router
    - Test successful request with valid whistleblower ID
    - Test 404 response for non-existent whistleblower
    - Test 400 response for invalid ID format
    - Test 500 response for service layer errors
    - _Requirements: 1.1, 1.2, 1.4_

- [ ] 5. Register router in app.ts
  - [x] 5.1 Wire whistleblower router into Express app
    - Import createWhistleblowerRouter factory
    - Instantiate earnings service with dependencies
    - Create router instance and mount at /api path
    - Place before 404 catch-all handler
    - _Requirements: 1.1_

- [ ] 6. Update OpenAPI specification
  - [x] 6.1 Document the earnings endpoint in openapi.yml
    - Add GET /api/whistleblower/{id}/earnings path
    - Document path parameters (id)
    - Document 200 response schema with totals and history
    - Document error responses (400, 404, 500)
    - Include example responses with different payout statuses
    - _Requirements: 4.1, 4.2, 4.3_

- [ ] 7. Final checkpoint - Code quality checks
  - [x] 7.1 Run linting and fix any errors
    - Execute npm run lint
    - Fix any linting errors
    - _Requirements: 5.1_
  - [x] 7.2 Run type checking and fix any errors
    - Execute npm run type-check (or tsc --noEmit)
    - Fix any type errors
    - _Requirements: 5.2_
  - [x] 7.3 Run build and verify success
    - Execute npm run build
    - Verify build completes without errors
    - _Requirements: 5.3_
  - [x] 7.4 Run all tests and verify they pass
    - Execute npm test
    - Ensure all unit tests and property tests pass
    - _Requirements: 1.2, 1.3, 1.4, 2.2, 2.3, 2.4, 2.5, 3.2, 3.3, 3.4, 3.5_

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Property-based tests use fast-check library with minimum 100 iterations
- The service layer assumes a data layer interface exists or will be mocked for testing
- Currency conversion rate (USDC to NGN) should be sourced from configuration or external service
- All error handling follows the existing AppError pattern established in the codebase

# Leave Dialog Optimization - TODO List

## Project Overview
Refactoring the monolithic LeaveDialog component (2200+ lines) into a modular, performant, and maintainable architecture.

## Current Status: Phase 1 - Immediate Fixes & Verification
**Last Updated:** 2025-09-10

---

## Phase 1: Immediate Fixes & Verification (1-2 days)
**Goal:** Ensure all current fixes work correctly and identify any remaining issues

### 1.1 Verification Tasks
- [ ] **Test Leave Type Transitions** (Priority: HIGH)
  - [ ] Holiday → WFH → Holiday transitions work correctly
  - [ ] Holiday → Maternity → Holiday transitions work correctly  
  - [ ] Holiday → Sick → Holiday transitions work correctly
  - [ ] WFH → Maternity → WFH transitions work correctly
  - [ ] No UI flickering or wrong leave type displays
  - [ ] Form data persists correctly during transitions

- [ ] **Test Edit Mode Scenarios** (Priority: HIGH)
  - [ ] Open Holiday leave edit → shows Holiday UI immediately
  - [ ] Open WFH leave edit → shows WFH UI immediately
  - [ ] Open Maternity leave edit → shows Maternity UI immediately
  - [ ] Open Sick leave edit → shows Sick UI immediately
  - [ ] No delay in showing correct leave type UI

- [ ] **Code Quality Verification** (Priority: MEDIUM)
  - [ ] Run `pnpm typecheck` - ensure no TypeScript errors
  - [ ] Run `pnpm lint` - ensure no linting issues
  - [ ] Run `pnpm build` - ensure successful build
  - [ ] Test in different browsers (Chrome, Firefox, Edge)

### 1.2 Bug Fixes (If Found During Testing)
- [ ] **Fix any remaining UI delay issues**
  - [ ] Document exact scenarios where delays occur
  - [ ] Implement targeted fixes
  - [ ] Verify fixes with repeated testing

- [ ] **Fix any leave type detection issues**
  - [ ] Ensure maternity leaves show maternity UI
  - [ ] Ensure WFH leaves show WFH UI
  - [ ] Ensure all other types work correctly

### 1.3 Documentation & Cleanup
- [ ] **Document Current Architecture**
  - [ ] Create ARCHITECTURE.md explaining current fixes
  - [ ] Document the key prop solution
  - [ ] Document state management patterns
  - [ ] Add code comments for complex logic sections

- [ ] **Performance Baseline**
  - [ ] Measure current render times with browser DevTools
  - [ ] Document bundle size before refactor
  - [ ] Create performance benchmarks for comparison

**Phase 1 Success Criteria:**
- ✅ All leave type transitions work instantly without UI delays
- ✅ No flickering or wrong UI displays
- ✅ All TypeScript/lint checks pass
- ✅ Documented current state for team reference

---

## Phase 2: Architecture Planning & Base Components (3-5 days)
**Goal:** Create foundation for modular architecture without breaking existing functionality

### 2.1 Architecture Design
- [ ] **Create Architecture Documentation**
  - [ ] Design component hierarchy diagram
  - [ ] Define shared vs specific component responsibilities
  - [ ] Plan state management strategy (Zustand slices)
  - [ ] Define TypeScript interfaces for each leave type

### 2.2 Extract Base Components
- [ ] **Create BaseLeaveDialog Component**
  - [ ] Extract common dialog layout (header, footer, buttons)
  - [ ] Extract shared date picker logic
  - [ ] Extract common form validation patterns
  - [ ] Extract shared API call patterns
  - [ ] Test with existing Holiday leave type

- [ ] **Create Shared Hooks**
  - [ ] `useBaseLeaveLogic` - common form handling
  - [ ] `useLeaveDateValidation` - date validation logic
  - [ ] `useLeaveAPI` - API call abstractions
  - [ ] `useLeaveErrorHandling` - error management

### 2.3 Type System Design
- [ ] **Define Shared Types**
  - [ ] `BaseLeaveFormData` interface
  - [ ] `BaseLeaveValidation` interface  
  - [ ] `LeaveDialogProps` interface
  - [ ] Leave type specific interfaces

- [ ] **Create Type Guards**
  - [ ] `isHolidayLeave()` type guard
  - [ ] `isWFHLeave()` type guard
  - [ ] `isSickLeave()` type guard
  - [ ] `isMaternityLeave()` type guard

**Phase 2 Success Criteria:**
- ✅ BaseLeaveDialog component working with Holiday leaves
- ✅ Shared hooks tested and documented
- ✅ Type system designed and validated
- ✅ No regression in existing functionality

---

## Phase 3: Individual Leave Type Components (1-2 weeks)
**Goal:** Create specialized components for each leave type

### 3.1 Holiday Leave Component (Days 1-2)
- [ ] **Create HolidayLeaveDialog Component**
  - [ ] Extract Holiday-specific form fields
  - [ ] Implement Holiday-specific validation
  - [ ] Create Holiday state management (Zustand slice)
  - [ ] Test Holiday functionality isolation

- [ ] **Holiday-Specific Features**
  - [ ] Half-day selection logic
  - [ ] Duration calculation
  - [ ] Holiday balance validation
  - [ ] Date range restrictions

### 3.2 WFH Leave Component (Days 3-4)
- [ ] **Create WFHLeaveDialog Component**
  - [ ] Extract WFH-specific form fields
  - [ ] Implement WFH time selection
  - [ ] Create WFH state management
  - [ ] Test WFH functionality

- [ ] **WFH-Specific Features**
  - [ ] Time range picker (from/to hours)
  - [ ] Duration type (hours/days)
  - [ ] WFH policy validation

### 3.3 Sick Leave Component (Days 5-7)
- [ ] **Create SickLeaveDialog Component**
  - [ ] Extract Sick-specific form fields
  - [ ] Implement complex Sick validations
  - [ ] Create Sick state management
  - [ ] Test Sick leave tabs functionality

- [ ] **Sick-Specific Features**
  - [ ] Emergency leave checkbox
  - [ ] Doctor consultation logic
  - [ ] Medical certificate handling
  - [ ] Return to work interview requirements

### 3.4 Maternity Leave Component (Days 8-10)
- [ ] **Create MaternityLeaveDialog Component**
  - [ ] Extract Maternity-specific form fields
  - [ ] Implement complex date calculations
  - [ ] Create Maternity state management
  - [ ] Test Maternity tabs functionality

- [ ] **Maternity-Specific Features**
  - [ ] Due date calculations
  - [ ] Leave period calculations
  - [ ] Statutory leave requirements
  - [ ] Document management

**Phase 3 Success Criteria:**
- ✅ All 4 specialized components working independently
- ✅ Each component <500 lines of code
- ✅ Type-safe validations for each leave type
- ✅ Comprehensive testing for each component

---

## Phase 4: Integration & Router Implementation (2-3 days)
**Goal:** Connect all components through a central router

### 4.1 Create Dialog Router
- [ ] **LeaveDialogContainer Component**
  - [ ] Implement leave type detection logic
  - [ ] Add lazy loading for each leave type component
  - [ ] Implement loading states and error boundaries
  - [ ] Add fallback components for errors

- [ ] **Route Management**
  - [ ] Dynamic component loading based on leave type
  - [ ] Proper props passing to specialized components
  - [ ] Error handling for unknown leave types

### 4.2 Integration Testing
- [ ] **End-to-End Testing**
  - [ ] Test all leave type switches through router
  - [ ] Verify lazy loading works correctly
  - [ ] Test error scenarios and fallbacks
  - [ ] Performance testing for component switching

- [ ] **Regression Testing**
  - [ ] Verify all original functionality still works
  - [ ] Test edge cases and error conditions
  - [ ] Validate form submissions for all leave types

### 4.3 Migration
- [ ] **Replace Original Component**
  - [ ] Update LeavePlanner to use LeaveDialogContainer
  - [ ] Remove old LeaveDialog references
  - [ ] Update imports and prop passing
  - [ ] Test complete integration

**Phase 4 Success Criteria:**
- ✅ Router correctly loads appropriate leave type component
- ✅ Lazy loading reduces initial bundle size
- ✅ All original functionality preserved
- ✅ Error handling works properly

---

## Phase 5: Global State Management (2-3 days)
**Goal:** Implement centralized state management for better performance

### 5.1 Zustand Store Setup
- [ ] **Create Leave Store Slices**
  - [ ] `holidayLeave.slice.ts` - Holiday leave state
  - [ ] `wfhLeave.slice.ts` - WFH leave state  
  - [ ] `sickLeave.slice.ts` - Sick leave state
  - [ ] `maternityLeave.slice.ts` - Maternity leave state

- [ ] **Global Leave Store**
  - [ ] Combine all leave type slices
  - [ ] Implement cross-slice communication
  - [ ] Add persistence for form data
  - [ ] Add caching for API responses

### 5.2 State Migration
- [ ] **Convert Components to Use Global State**
  - [ ] Update HolidayLeaveDialog to use store
  - [ ] Update WFHLeaveDialog to use store
  - [ ] Update SickLeaveDialog to use store
  - [ ] Update MaternityLeaveDialog to use store

- [ ] **Remove Local State**
  - [ ] Replace useState with store actions
  - [ ] Remove prop drilling
  - [ ] Simplify component hierarchies

### 5.3 Performance Optimization
- [ ] **Implement Smart Updates**
  - [ ] Selective re-renders with store selectors
  - [ ] Debounced API calls
  - [ ] Optimistic updates for better UX

**Phase 5 Success Criteria:**
- ✅ Global state management working correctly
- ✅ Improved performance with selective updates
- ✅ Form data persistence across navigation
- ✅ Reduced prop drilling and component complexity

---

## Phase 6: Advanced Optimizations (1-2 weeks)
**Goal:** Final performance optimizations and polish

### 6.1 Code Splitting & Bundle Optimization
- [ ] **Implement Advanced Lazy Loading**
  - [ ] Route-based code splitting
  - [ ] Preload likely-needed components
  - [ ] Implement bundle analysis
  - [ ] Optimize chunk sizes

### 6.2 Performance Enhancements
- [ ] **Micro-optimizations**
  - [ ] Virtual scrolling for large dropdowns
  - [ ] Memoization of expensive calculations
  - [ ] Debounced API calls and validation
  - [ ] Image lazy loading and optimization

### 6.3 Developer Experience
- [ ] **Enhanced Tooling**
  - [ ] Create component templates/generators
  - [ ] Add Storybook stories for each component
  - [ ] Create comprehensive test suite
  - [ ] Add performance monitoring

### 6.4 Documentation & Training
- [ ] **Team Documentation**
  - [ ] Create developer guide for adding new leave types
  - [ ] Document state management patterns
  - [ ] Create troubleshooting guide
  - [ ] Record demo videos for team training

**Phase 6 Success Criteria:**
- ✅ 60-80% reduction in initial bundle size
- ✅ Sub-100ms component switching times
- ✅ Comprehensive documentation and tooling
- ✅ Team trained on new architecture

---

## Guidelines & Best Practices

### Development Guidelines
1. **Always test before moving to next phase**
2. **Maintain backward compatibility during migration**  
3. **Write TypeScript-first, ensure type safety**
4. **Follow existing code style and patterns**
5. **Document architectural decisions**

### Testing Strategy
- Unit tests for individual components
- Integration tests for component interactions
- E2E tests for critical user flows
- Performance regression tests
- Cross-browser compatibility testing

### Performance Targets
- **Initial Bundle Size:** <50KB per leave type component
- **Component Switch Time:** <100ms
- **Form Render Time:** <200ms
- **API Response Handling:** <300ms

### Code Quality Standards
- **TypeScript strict mode enabled**
- **ESLint + Prettier compliance**
- **100% type coverage for new components**
- **Comprehensive error handling**
- **Accessibility compliance (WCAG 2.1)**

---

## Current Progress Tracking

### Phase 1 Progress: 85% Complete ✅
- ✅ Fixed maternity leave type detection
- ✅ Eliminated UI flickering 
- ✅ Fixed leave type transition delays
- ✅ Added component key prop for clean state
- ⏳ **Currently Testing:** All transition scenarios
- 🔄 **Next:** Code quality verification

### Immediate Next Actions:
1. **Test all leave type transitions thoroughly**
2. **Run typecheck and lint verification**  
3. **Document current fixes for team**
4. **Plan Phase 2 architecture design**

---

## Risk Management

### Identified Risks:
1. **Breaking existing functionality during refactor**
   - Mitigation: Comprehensive testing at each phase
   
2. **Team adoption of new architecture**
   - Mitigation: Gradual migration + training sessions

3. **Performance regression during migration**
   - Mitigation: Continuous performance monitoring

### Rollback Strategy:
- Keep original LeaveDialog as backup until Phase 4 completion
- Feature flags for new vs old components
- Database/API compatibility maintained throughout

---

*This TODO list is a living document. Update task status and add new insights as development progresses.*

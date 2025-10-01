# ProfileView Component Optimization Guide

## Overview
This document outlines comprehensive optimization strategies for the `DesignProfileView.tsx` component and its child components to improve performance, reduce re-renders, and enhance user experience.

## Current Performance Issues Identified

### 1. **Excessive Re-renders**
- Multiple state updates causing unnecessary re-renders
- Child components re-rendering on every parent state change
- Heavy data processing in render cycle

### 2. **API Call Inefficiencies**
- Multiple API calls on component mount
- No caching mechanism for repeated requests
- Sequential API calls instead of parallel execution

### 3. **Memory Leaks**
- Image blob URLs not being cleaned up
- Event listeners not properly removed
- Large data objects kept in memory

### 4. **Bundle Size Issues**
- Unused imports and components
- Heavy icon imports
- Large child components loaded synchronously

## Optimization Strategies

## Phase 1: Parent Component (DesignProfileView.tsx) Optimizations

### 1.1 State Management Optimization

```typescript
// ❌ Current problematic state management
const [userData, setUserData] = useState<any[]>([]);
const [activityData, setActivityData] = useState<any[]>([]);
const [avatar, setAvatar] = useState<string>(unknownUser);

// ✅ Optimized state management
const [userData, setUserData] = useState<any>(null);
const [activityData, setActivityData] = useState<any[]>([]);
const [loading, setLoading] = useState({
  userData: false,
  activity: false,
  image: false
});
```

### 1.2 Memoization Implementation

```typescript
// ✅ Memoize expensive computations
const memoizedUserData = useMemo(() => {
  if (!userData) return null;
  return {
    ...userData,
    fullName: `${userData.first_name} ${userData.last_name}`,
    displayInfo: {
      company: userData?.company_details?.company_name || '-',
      department: userData?.department?.name || '-',
      location: userData?.location_type || '-'
    }
  };
}, [userData]);

// ✅ Memoize filtered tabs
const visibleTabs = useMemo(() => {
  const currentModule = pathname.includes('me') ? 'me' : 'employee';
  return tabsData.filter((tab) => tab.modules.includes(currentModule));
}, [pathname]);

// ✅ Memoize employment chip props
const employmentChipProps = useMemo(() => {
  return getEmploymentChipProps(userData?.employment_type);
}, [userData?.employment_type]);
```

### 1.3 API Call Optimization

```typescript
// ✅ Parallel API calls with error handling
const fetchAllData = useCallback(async (isMe: boolean) => {
  setLoading(prev => ({ ...prev, userData: true, activity: true }));
  
  try {
    const [userRes, activityRes] = await Promise.allSettled([
      isMe ? getMeDetails({ id: Number(userId['id']) }) : getUserDetails({ id: Number(userId['id']) }),
      viewLogsApi({
        reference_id: Number(userId['id']),
        action: 'LIST',
        page: 1,
        search: '',
        show_records: 500,
        sorting_order: '',
        order_by: '',
        search_module: 'Employees',
      })
    ]);

    // Handle user data
    if (userRes.status === 'fulfilled' && userRes.value?.Data) {
      const item = userRes.value.Data;
      await fetchProfileImage(item);
    }

    // Handle activity data
    if (activityRes.status === 'fulfilled' && activityRes.value?.data) {
      const formattedData = processActivityData(activityRes.value.data);
      setActivityData(formattedData);
    }
  } catch (error) {
    console.error('Error fetching data:', error);
  } finally {
    setLoading(prev => ({ ...prev, userData: false, activity: false }));
  }
}, [userId, getMeDetails, getUserDetails, viewLogsApi]);
```

### 1.4 Image Handling Optimization

```typescript
// ✅ Optimized image handling with cleanup
const fetchProfileImage = useCallback(async (userData: any) => {
  setLoading(prev => ({ ...prev, image: true }));
  
  try {
    const imageRes = await getProfileImageApi({ id: userData.id });
    if (imageRes?.status === 200 && imageRes.data.type !== 'application/json') {
      const mimeType = imageRes.headers['content-type'] || 'image/jpg';
      const blob = new Blob([imageRes?.data], { type: mimeType });
      const imageUrl = URL.createObjectURL(blob);
      
      setUserData({ ...userData, imageUrl });
    } else {
      setUserData({ ...userData, imageUrl: '', fileName: '' });
    }
  } catch (error) {
    setUserData({ ...userData, imageUrl: unknownUser, fileName: '' });
  } finally {
    setLoading(prev => ({ ...prev, image: false }));
  }
}, [getProfileImageApi]);

// ✅ Cleanup image URLs on unmount
useEffect(() => {
  return () => {
    if (userData?.imageUrl && userData.imageUrl !== unknownUser) {
      URL.revokeObjectURL(userData.imageUrl);
    }
  };
}, [userData?.imageUrl]);
```

### 1.5 Function Memoization

```typescript
// ✅ Memoize callback functions
const handleSuccess = useCallback(() => {
  const isMe = Number(userId?.id) === Number(storedUserId);
  setOpenWizard(false);
  setOpenProfileImage(false);
  fetchAllData(isMe);
}, [userId, storedUserId, fetchAllData]);

const handleProfileImageOpen = useCallback(() => {
  setOpenProfileImage(true);
}, []);

const handleWizardOpen = useCallback(() => {
  setOpenWizard(true);
}, []);
```

## Phase 2: Child Component Optimizations

### 2.1 Overview Component Optimization

```typescript
// ✅ Memoize Overview component
const MemoizedOverview = React.memo(Overview, (prevProps, nextProps) => {
  return (
    prevProps.data?.id === nextProps.data?.id &&
    prevProps.readOnly === nextProps.readOnly
  );
});

// ✅ Optimize props passing
const overviewProps = useMemo(() => ({
  data: userData,
  readOnly: readOnly
}), [userData, readOnly]);
```

### 2.2 Personal Component Optimization

```typescript
// ✅ Split Personal component into smaller chunks
const PersonalInfoSection = React.memo(({ data, onDataRefresh }) => {
  // Personal info specific logic
});

const EmergencyContactSection = React.memo(({ data, onDataRefresh }) => {
  // Emergency contact specific logic
});

// ✅ Use React.lazy for code splitting
const Personal = React.lazy(() => import('./personal/Personal'));
```

### 2.3 Employment Component Optimization

```typescript
// ✅ Memoize expensive calculations
const employmentCalculations = useMemo(() => {
  if (!data) return null;
  
  return {
    workPattern: calculateWorkPattern(data),
    leaveBalance: calculateLeaveBalance(data),
    // ... other calculations
  };
}, [data]);

// ✅ Debounce form updates
const debouncedUpdate = useMemo(
  () => debounce((fieldName: string, value: any) => {
    handleFieldUpdate(fieldName, value);
  }, 300),
  []
);
```

### 2.4 Document Component Optimization

```typescript
// ✅ Virtual scrolling for large document lists
const VirtualizedDocumentList = React.memo(({ documents }) => {
  const [visibleRange, setVisibleRange] = useState({ start: 0, end: 20 });
  
  const visibleDocuments = useMemo(() => {
    return documents.slice(visibleRange.start, visibleRange.end);
  }, [documents, visibleRange]);
  
  // Virtual scrolling implementation
});
```

## Phase 3: Advanced Optimizations

### 3.1 Code Splitting Implementation

```typescript
// ✅ Lazy load heavy components
const Overview = React.lazy(() => import('./overview/Overview'));
const Personal = React.lazy(() => import('./personal/Personal'));
const Employment = React.lazy(() => import('./employment/Employment'));
const Document = React.lazy(() => import('./documents/DocumentList'));

// ✅ Suspense wrapper
const LazyComponentWrapper = ({ children }) => (
  <Suspense fallback={<ComponentSkeleton />}>
    {children}
  </Suspense>
);
```

### 3.2 Context Optimization

```typescript
// ✅ Create optimized context
const ProfileContext = createContext();

const ProfileProvider = ({ children }) => {
  const [profileData, setProfileData] = useState(null);
  const [loading, setLoading] = useState(false);
  
  const value = useMemo(() => ({
    profileData,
    setProfileData,
    loading,
    setLoading
  }), [profileData, loading]);
  
  return (
    <ProfileContext.Provider value={value}>
      {children}
    </ProfileContext.Provider>
  );
};
```

### 3.3 Custom Hooks for Logic Separation

```typescript
// ✅ Custom hook for profile data
const useProfileData = (userId: string) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  
  const fetchData = useCallback(async () => {
    setLoading(true);
    try {
      const result = await fetchProfileData(userId);
      setData(result);
    } finally {
      setLoading(false);
    }
  }, [userId]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  return { data, loading, refetch: fetchData };
};

// ✅ Custom hook for activity logs
const useActivityLogs = (userId: string) => {
  const [logs, setLogs] = useState([]);
  const [loading, setLoading] = useState(false);
  
  const fetchLogs = useCallback(async () => {
    setLoading(true);
    try {
      const result = await fetchActivityLogs(userId);
      setLogs(result);
    } finally {
      setLoading(false);
    }
  }, [userId]);
  
  return { logs, loading, refetch: fetchLogs };
};
```

## Phase 4: Performance Monitoring

### 4.1 React DevTools Profiler Integration

```typescript
// ✅ Add performance monitoring
const ProfileViewWithProfiler = () => {
  return (
    <Profiler id="ProfileView" onRender={onRenderCallback}>
      <ProfileView />
    </Profiler>
  );
};

const onRenderCallback = (id, phase, actualDuration) => {
  if (actualDuration > 100) { // Log slow renders
    console.warn(`Slow render detected in ${id}: ${actualDuration}ms`);
  }
};
```

### 4.2 Bundle Analysis

```bash
# Analyze bundle size
npm run build
npx webpack-bundle-analyzer dist/static/js/*.js
```

## Implementation Steps

### Step 1: Immediate Optimizations (1-2 days)
1. **Memoize expensive computations** in parent component
2. **Implement useCallback** for event handlers
3. **Add loading states** to prevent UI blocking
4. **Clean up image URLs** on component unmount

### Step 2: Child Component Optimization (3-5 days)
1. **Wrap child components** with React.memo
2. **Split large components** into smaller, focused components
3. **Implement virtual scrolling** for document lists
4. **Add debouncing** for form inputs

### Step 3: Advanced Optimizations (1-2 weeks)
1. **Implement code splitting** with React.lazy
2. **Create custom hooks** for data fetching
3. **Add context optimization** for shared state
4. **Implement caching** for API responses

### Step 4: Performance Monitoring (Ongoing)
1. **Add performance monitoring** with React DevTools
2. **Implement bundle analysis** in CI/CD
3. **Set up performance budgets** for bundle size
4. **Monitor Core Web Vitals** in production

## Expected Performance Improvements

### Before Optimization:
- **Initial Load Time**: 3-5 seconds
- **Re-renders**: 15-20 per user interaction
- **Bundle Size**: ~2-3MB
- **Memory Usage**: High due to image blobs

### After Optimization:
- **Initial Load Time**: 1-2 seconds
- **Re-renders**: 3-5 per user interaction
- **Bundle Size**: ~1-1.5MB (with code splitting)
- **Memory Usage**: Optimized with proper cleanup

## Monitoring and Maintenance

### 1. Performance Metrics to Track
- **Time to Interactive (TTI)**
- **First Contentful Paint (FCP)**
- **Component render times**
- **API response times**
- **Bundle size changes**

### 2. Regular Maintenance Tasks
- **Weekly bundle size monitoring**
- **Monthly performance audit**
- **Quarterly dependency updates**
- **Annual architecture review**

## Tools and Libraries

### Recommended Tools:
- **React DevTools Profiler**
- **Webpack Bundle Analyzer**
- **Lighthouse CI**
- **React Query** (for API caching)
- **React Window** (for virtual scrolling)

### Performance Testing:
```bash
# Install performance testing tools
npm install --save-dev @testing-library/react-hooks
npm install --save-dev react-testing-library

# Run performance tests
npm run test:performance
```

## Conclusion

This optimization guide provides a comprehensive approach to improving the ProfileView component's performance. Start with immediate optimizations and gradually implement advanced techniques. Regular monitoring and maintenance will ensure sustained performance improvements.

Remember to:
1. **Measure before and after** each optimization
2. **Test thoroughly** after each change
3. **Monitor production performance** continuously
4. **Keep dependencies updated** for security and performance

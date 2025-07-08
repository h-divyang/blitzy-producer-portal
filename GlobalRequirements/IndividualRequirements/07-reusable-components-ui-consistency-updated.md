# 7.0 Reusable Components & UI Consistency - Updated

## Frontend Framework Architecture

### React 18+ with TypeScript
- **Framework**: React 18+ for modern features and performance
- **Language**: TypeScript for type safety and developer experience
- **Build Tool**: Vite for fast development and optimized builds
- **Module System**: ES modules with tree shaking
- **Development**: Hot module replacement for rapid iteration

### Minimal UI Kit Integration
- **Purpose**: Comprehensive design system and component library
- **Version**: Latest version with React 18 support
- **Components**: 50+ pre-built components covering all UI needs
- **Customization**: Theming system with design tokens
- **Documentation**: Storybook integration for component docs

### Tailwind CSS Framework
- **Version**: Tailwind CSS 3.x for modern utility classes
- **Configuration**: Custom configuration extending default theme
- **Plugins**: Forms, typography, aspect-ratio plugins
- **Performance**: PurgeCSS for production optimization
- **Integration**: PostCSS pipeline with autoprefixer

## Component Architecture

### Atomic Design Principles
- **Atoms**: Basic UI elements (buttons, inputs, labels)
- **Molecules**: Simple component groups (form fields, cards)
- **Organisms**: Complex components (forms, navigation)
- **Templates**: Page layouts and structures
- **Pages**: Complete page implementations

### Component Organization
```
src/
├── components/
│   ├── atoms/
│   │   ├── Button/
│   │   ├── Input/
│   │   └── Label/
│   ├── molecules/
│   │   ├── FormField/
│   │   ├── Card/
│   │   └── Alert/
│   ├── organisms/
│   │   ├── PolicyForm/
│   │   ├── ClaimsTable/
│   │   └── Navigation/
│   └── templates/
│       ├── DashboardLayout/
│       ├── AuthLayout/
│       └── PublicLayout/
```

## Design System

### Design Tokens
```typescript
// tokens/colors.ts
export const colors = {
  primary: {
    50: '#eff6ff',
    500: '#3b82f6',
    900: '#1e3a8a',
  },
  semantic: {
    success: '#10b981',
    warning: '#f59e0b',
    error: '#ef4444',
    info: '#3b82f6',
  },
  // Client-specific theme overrides
};
```

### Typography System
- **Font Families**: 
  - Primary: Inter for UI elements
  - Secondary: SF Mono for code/data
- **Type Scale**: Consistent sizing with rem units
- **Line Heights**: Optimized for readability
- **Font Weights**: 400 (regular), 500 (medium), 600 (semibold), 700 (bold)

### Spacing System
- **Base Unit**: 4px grid system
- **Scale**: 0, 1, 2, 4, 8, 12, 16, 20, 24, 32, 40, 48, 56, 64
- **Consistent Application**: Padding, margin, gap utilities
- **Responsive**: Mobile-first responsive spacing

## Core UI Components

### Form Components
```typescript
// PolicyForm Component
interface PolicyFormProps {
  initialData?: PolicyData;
  onSubmit: (data: PolicyData) => Promise<void>;
  mode: 'create' | 'edit' | 'quote';
}

// Reusable form fields with validation
<FormField
  label="Policy Number"
  name="policyNumber"
  rules={{ required: true, pattern: /^POL-\d{6}$/ }}
  error={errors.policyNumber}
/>
```

### Data Display Components
- **DataTable**: Sortable, filterable, paginated tables
- **Card**: Consistent card layouts for data grouping
- **Stat**: Dashboard statistics display
- **Timeline**: Claims/policy history visualization
- **Charts**: Recharts integration for data visualization

### Navigation Components
- **TopNav**: Primary navigation with user menu
- **SideNav**: Collapsible sidebar navigation
- **Breadcrumb**: Hierarchical navigation
- **TabNav**: In-page navigation
- **MobileNav**: Responsive mobile navigation

### Feedback Components
- **Alert**: Success, warning, error, info states
- **Toast**: Non-blocking notifications
- **Modal**: Accessible modal dialogs
- **Drawer**: Slide-out panels
- **Progress**: Loading and progress indicators

## Insurance-Specific Components

### Policy Components
- **PolicyCard**: Policy summary display
- **PolicyTimeline**: Policy lifecycle visualization
- **CoverageSelector**: Dynamic coverage selection
- **PremiumCalculator**: Real-time premium display
- **DocumentViewer**: Policy document display

### Claims Components
- **ClaimForm**: Multi-step claim submission
- **ClaimStatus**: Visual claim status tracker
- **DocumentUpload**: Drag-and-drop file upload
- **ClaimTimeline**: Claim progress visualization
- **SettlementSummary**: Payment breakdown display

### Quote Components
- **QuoteWizard**: Multi-step quote process
- **RiskAssessment**: Dynamic risk factor inputs
- **QuoteComparison**: Side-by-side quote display
- **BindButton**: Quote-to-policy conversion
- **QuoteSummary**: Comprehensive quote details

## Theme System

### Theme Provider
```typescript
// Theme configuration
const theme = {
  colors: { ...designTokens.colors },
  typography: { ...designTokens.typography },
  spacing: { ...designTokens.spacing },
  components: {
    Button: {
      variants: ['primary', 'secondary', 'danger'],
      sizes: ['sm', 'md', 'lg'],
    },
  },
};

// Multi-tenant theming
const getClientTheme = (clientId: string) => ({
  ...baseTheme,
  ...clientThemeOverrides[clientId],
});
```

### Dark Mode Support
- **System Preference**: Automatic dark mode detection
- **User Preference**: Manual toggle with persistence
- **Component Support**: All components dark mode compatible
- **Custom Properties**: CSS variables for dynamic theming

## Component Development Standards

### TypeScript Standards
```typescript
// Strict type definitions
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  disabled?: boolean;
  onClick?: (event: React.MouseEvent) => void;
  children: React.ReactNode;
}

// Proper component typing
const Button: React.FC<ButtonProps> = ({ 
  variant = 'primary',
  size = 'md',
  ...props 
}) => {
  // Implementation
};
```

### Accessibility Standards
- **WCAG 2.1 AA Compliance**: All components meet standards
- **Keyboard Navigation**: Full keyboard support
- **Screen Reader**: Proper ARIA labels and roles
- **Focus Management**: Visible focus indicators
- **Color Contrast**: Meets contrast requirements

### Performance Standards
- **Code Splitting**: Dynamic imports for large components
- **Memoization**: React.memo for expensive components
- **Lazy Loading**: Images and heavy components
- **Bundle Size**: Monitor and optimize bundle size
- **Render Optimization**: Minimize unnecessary renders

## Testing Strategy

### Component Testing
```typescript
// Unit tests with React Testing Library
describe('PolicyForm', () => {
  it('validates required fields', async () => {
    const { getByRole, getByText } = render(<PolicyForm />);
    const submitButton = getByRole('button', { name: /submit/i });
    
    fireEvent.click(submitButton);
    
    await waitFor(() => {
      expect(getByText(/policy number is required/i)).toBeInTheDocument();
    });
  });
});
```

### Visual Regression Testing
- **Chromatic**: Automated visual testing
- **Storybook**: Component documentation and testing
- **Percy**: Cross-browser visual testing
- **Snapshot Testing**: Jest snapshot tests

## Documentation

### Component Documentation
- **Storybook Stories**: Interactive component examples
- **Props Documentation**: Auto-generated from TypeScript
- **Usage Examples**: Real-world implementation examples
- **Design Guidelines**: When and how to use components
- **Accessibility Notes**: Component-specific a11y considerations

### Style Guide
- **Component Naming**: PascalCase for components
- **File Structure**: Component folder with index, styles, tests
- **Import Order**: Consistent import organization
- **Code Formatting**: Prettier configuration
- **Linting Rules**: ESLint with React/TypeScript rules

## Responsive Design

### Breakpoint System
```scss
// Tailwind default breakpoints
sm: 640px   // Mobile landscape
md: 768px   // Tablet portrait
lg: 1024px  // Tablet landscape
xl: 1280px  // Desktop
2xl: 1536px // Large desktop
```

### Mobile-First Approach
- **Default Styles**: Mobile styles as base
- **Progressive Enhancement**: Add complexity for larger screens
- **Touch Optimization**: Larger tap targets on mobile
- **Performance**: Optimize assets for mobile networks
- **Testing**: Test on real devices and browsers

## Integration with Backend

### API Integration Patterns
```typescript
// Custom hooks for data fetching
const usePolicyData = (policyId: string) => {
  return useQuery({
    queryKey: ['policy', policyId],
    queryFn: () => api.getPolicy(policyId),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
};

// Optimistic updates
const updatePolicy = useMutation({
  mutationFn: api.updatePolicy,
  onMutate: async (newData) => {
    // Optimistically update UI
  },
});
```

### State Management Integration
- **React Query**: Server state management
- **Context API**: Client state management
- **Form State**: React Hook Form integration
- **Error Boundaries**: Graceful error handling
- **Loading States**: Skeleton screens and spinners
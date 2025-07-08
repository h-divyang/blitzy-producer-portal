# 11.0 Accessibility - Updated

## Accessibility Framework

### WCAG 2.1 AA Compliance
- **Standard**: Web Content Accessibility Guidelines 2.1 Level AA compliance
- **Legal Requirements**: ADA compliance for US operations
- **Insurance Industry**: Enhanced accessibility for diverse user populations including elderly users
- **Testing**: Automated and manual accessibility validation

### Accessibility-First Design Philosophy
- **Inclusive Design**: Design for the widest range of users from the start
- **Progressive Enhancement**: Build accessible foundation, enhance with advanced features
- **User Testing**: Regular testing with users who have disabilities
- **Accessibility Champions**: Dedicated team members for accessibility advocacy

## React Component Accessibility

### Semantic HTML Foundation
```typescript
// Semantic structure example
const PolicyForm: React.FC = () => {
  return (
    <main role="main" aria-labelledby="policy-form-title">
      <h1 id="policy-form-title">Create New Policy</h1>
      <form onSubmit={handleSubmit} noValidate>
        <fieldset>
          <legend>Insured Information</legend>
          <div className="form-group">
            <label htmlFor="insured-name" className="required">
              Insured Name
            </label>
            <input
              id="insured-name"
              type="text"
              required
              aria-describedby="insured-name-help insured-name-error"
              aria-invalid={errors.insuredName ? 'true' : 'false'}
            />
            <div id="insured-name-help" className="help-text">
              Enter the full legal name of the insured party
            </div>
            {errors.insuredName && (
              <div id="insured-name-error" className="error-message" role="alert">
                {errors.insuredName.message}
              </div>
            )}
          </div>
        </fieldset>
      </form>
    </main>
  );
};
```

### ARIA Implementation
- **ARIA Labels**: Descriptive labels for all interactive elements
- **ARIA Roles**: Semantic roles for custom components
- **ARIA States**: Dynamic state communication (expanded, selected, busy)
- **ARIA Descriptions**: Additional context for complex interactions
- **Live Regions**: Announcements for dynamic content changes

### Form Accessibility
```typescript
// Accessible form field component
interface FormFieldProps {
  label: string;
  name: string;
  type?: string;
  required?: boolean;
  helpText?: string;
  error?: string;
  description?: string;
}

const FormField: React.FC<FormFieldProps> = ({
  label,
  name,
  required,
  helpText,
  error,
  description,
  ...props
}) => {
  const fieldId = `field-${name}`;
  const helpId = `${fieldId}-help`;
  const errorId = `${fieldId}-error`;
  const descriptionId = `${fieldId}-description`;
  
  const describedBy = [
    description && descriptionId,
    helpText && helpId,
    error && errorId,
  ].filter(Boolean).join(' ');

  return (
    <div className="form-field">
      <label htmlFor={fieldId} className={required ? 'required' : ''}>
        {label}
        {required && <span aria-label="required"> *</span>}
      </label>
      
      {description && (
        <div id={descriptionId} className="field-description">
          {description}
        </div>
      )}
      
      <input
        id={fieldId}
        name={name}
        aria-describedby={describedBy || undefined}
        aria-invalid={error ? 'true' : 'false'}
        aria-required={required}
        {...props}
      />
      
      {helpText && (
        <div id={helpId} className="help-text">
          {helpText}
        </div>
      )}
      
      {error && (
        <div id={errorId} className="error-message" role="alert">
          {error}
        </div>
      )}
    </div>
  );
};
```

## Keyboard Navigation

### Focus Management
```typescript
// Custom hook for focus management
const useFocusManagement = () => {
  const focusRef = useRef<HTMLElement>(null);
  
  const setFocus = useCallback(() => {
    if (focusRef.current) {
      focusRef.current.focus();
    }
  }, []);
  
  const trapFocus = useCallback((event: KeyboardEvent) => {
    if (event.key === 'Tab') {
      const focusableElements = focusRef.current?.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      
      if (focusableElements) {
        const firstElement = focusableElements[0] as HTMLElement;
        const lastElement = focusableElements[focusableElements.length - 1] as HTMLElement;
        
        if (event.shiftKey && document.activeElement === firstElement) {
          event.preventDefault();
          lastElement.focus();
        } else if (!event.shiftKey && document.activeElement === lastElement) {
          event.preventDefault();
          firstElement.focus();
        }
      }
    }
  }, []);
  
  return { focusRef, setFocus, trapFocus };
};
```

### Tab Order Optimization
- **Logical Flow**: Tab order follows visual layout and workflow
- **Skip Links**: Skip to main content and navigation
- **Focus Indicators**: Visible focus indicators for all interactive elements
- **Modal Focus**: Focus trapping in dialogs and modals
- **Custom Components**: Proper keyboard event handling

### Keyboard Shortcuts
```typescript
// Keyboard shortcut implementation
const useKeyboardShortcuts = () => {
  useEffect(() => {
    const handleKeyPress = (event: KeyboardEvent) => {
      // Alt + N: New policy
      if (event.altKey && event.key === 'n') {
        event.preventDefault();
        navigate('/policies/new');
      }
      
      // Alt + S: Search
      if (event.altKey && event.key === 's') {
        event.preventDefault();
        document.getElementById('search-input')?.focus();
      }
      
      // Escape: Close modals
      if (event.key === 'Escape') {
        // Close any open modals
        closeAllModals();
      }
    };
    
    document.addEventListener('keydown', handleKeyPress);
    return () => document.removeEventListener('keydown', handleKeyPress);
  }, [navigate]);
};
```

## Screen Reader Support

### Screen Reader Testing
- **Primary Testing**: NVDA (Windows), VoiceOver (macOS), JAWS (Windows)
- **Mobile Testing**: VoiceOver (iOS), TalkBack (Android)
- **Content Structure**: Proper heading hierarchy and landmark usage
- **Table Accessibility**: Headers, captions, and summary for data tables

### Content Accessibility
```typescript
// Screen reader optimized table
const PolicyTable: React.FC<{ policies: Policy[] }> = ({ policies }) => {
  return (
    <table role="table" aria-label="Insurance Policies">
      <caption className="sr-only">
        List of {policies.length} insurance policies with their details
      </caption>
      <thead>
        <tr>
          <th scope="col" aria-sort="none">
            <button 
              aria-label="Sort by policy number"
              onClick={() => handleSort('number')}
            >
              Policy Number
            </button>
          </th>
          <th scope="col">Insured Name</th>
          <th scope="col">Premium</th>
          <th scope="col">Status</th>
          <th scope="col">Actions</th>
        </tr>
      </thead>
      <tbody>
        {policies.map((policy) => (
          <tr key={policy.id}>
            <td>{policy.number}</td>
            <td>{policy.insuredName}</td>
            <td>
              <span aria-label={`Premium ${policy.premium} dollars`}>
                ${policy.premium.toLocaleString()}
              </span>
            </td>
            <td>
              <span className={`status status-${policy.status.toLowerCase()}`}>
                {policy.status}
              </span>
            </td>
            <td>
              <button
                aria-label={`View details for policy ${policy.number}`}
                onClick={() => viewPolicy(policy.id)}
              >
                View
              </button>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
};
```

### Live Regions for Dynamic Content
```typescript
// Live region for status updates
const StatusAnnouncer: React.FC = () => {
  const [message, setMessage] = useState('');
  
  const announceStatus = useCallback((newMessage: string) => {
    setMessage(newMessage);
    // Clear message after announcement to allow re-announcement
    setTimeout(() => setMessage(''), 1000);
  }, []);
  
  return (
    <>
      <div
        aria-live="polite"
        aria-atomic="true"
        className="sr-only"
      >
        {message}
      </div>
      <div
        aria-live="assertive"
        aria-atomic="true"
        className="sr-only"
        id="urgent-announcements"
      />
    </>
  );
};
```

## Visual Accessibility

### Color and Contrast
```css
/* Tailwind CSS custom colors for WCAG AA compliance */
:root {
  /* Primary colors with 4.5:1 contrast ratio */
  --color-primary-50: #eff6ff;
  --color-primary-600: #2563eb; /* 4.5:1 on white */
  --color-primary-700: #1d4ed8; /* 7:1 on white */
  
  /* Error colors */
  --color-error-600: #dc2626; /* 4.5:1 on white */
  --color-error-700: #b91c1c; /* 7:1 on white */
  
  /* Success colors */
  --color-success-600: #16a34a; /* 4.5:1 on white */
  --color-success-700: #15803d; /* 7:1 on white */
}

/* Focus indicators */
.focus-visible {
  @apply outline-2 outline-offset-2 outline-blue-600;
}

/* High contrast mode support */
@media (prefers-contrast: high) {
  .btn-primary {
    border: 2px solid;
  }
}
```

### Typography and Readability
- **Font Size**: Minimum 16px base font size
- **Line Height**: 1.5 minimum for body text
- **Font Choice**: Sans-serif fonts for better readability
- **Text Spacing**: Adequate spacing between letters, words, and lines
- **Reading Width**: Optimal line length (45-75 characters)

### Responsive Design for Accessibility
```css
/* Zoom and magnification support */
@media (max-width: 1200px) {
  /* Support up to 200% zoom */
  .container {
    max-width: none;
    padding: 1rem;
  }
}

/* Reduced motion preferences */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* Dark mode support */
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #1f2937;
    --color-text: #f9fafb;
    --color-border: #374151;
  }
}
```

## Insurance-Specific Accessibility

### Complex Data Presentation
```typescript
// Accessible insurance quote comparison
const QuoteComparison: React.FC<{ quotes: Quote[] }> = ({ quotes }) => {
  return (
    <div role="region" aria-labelledby="quote-comparison-title">
      <h2 id="quote-comparison-title">Insurance Quote Comparison</h2>
      <div className="quote-grid" role="grid" aria-label="Quote comparison table">
        <div role="row" className="quote-header">
          <div role="columnheader">Carrier</div>
          <div role="columnheader">Premium</div>
          <div role="columnheader">Deductible</div>
          <div role="columnheader">Coverage Limits</div>
        </div>
        {quotes.map((quote, index) => (
          <div key={quote.id} role="row" className="quote-row">
            <div role="gridcell">
              <strong>{quote.carrier}</strong>
            </div>
            <div role="gridcell">
              <span aria-label={`Annual premium ${quote.premium} dollars`}>
                ${quote.premium}/year
              </span>
            </div>
            <div role="gridcell">
              <span aria-label={`Deductible ${quote.deductible} dollars`}>
                ${quote.deductible}
              </span>
            </div>
            <div role="gridcell">
              <ul aria-label="Coverage limits">
                {quote.coverages.map((coverage) => (
                  <li key={coverage.type}>
                    <span aria-label={`${coverage.type} coverage limit ${coverage.limit}`}>
                      {coverage.type}: ${coverage.limit.toLocaleString()}
                    </span>
                  </li>
                ))}
              </ul>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
};
```

### Form Complexity Management
- **Multi-Step Forms**: Clear progress indication and navigation
- **Conditional Fields**: Announce field visibility changes
- **Complex Calculations**: Real-time calculation announcements
- **Document Upload**: Accessible file upload with progress and status

## Automated Accessibility Testing

### Testing Integration
```typescript
// Jest + axe-core integration
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

describe('PolicyForm Accessibility', () => {
  it('should have no accessibility violations', async () => {
    const { container } = render(<PolicyForm />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
  
  it('should handle keyboard navigation', async () => {
    render(<PolicyForm />);
    
    // Tab through form elements
    await userEvent.tab();
    expect(screen.getByLabelText(/insured name/i)).toHaveFocus();
    
    await userEvent.tab();
    expect(screen.getByLabelText(/policy type/i)).toHaveFocus();
  });
  
  it('should announce form errors', async () => {
    render(<PolicyForm />);
    
    const submitButton = screen.getByRole('button', { name: /submit/i });
    await userEvent.click(submitButton);
    
    const errorMessage = await screen.findByRole('alert');
    expect(errorMessage).toBeInTheDocument();
  });
});
```

### Continuous Accessibility Monitoring
```yaml
# GitLab CI accessibility testing
accessibility-test:
  stage: test
  script:
    - npm ci
    - npm run build
    - npm run test:a11y
  artifacts:
    reports:
      accessibility: accessibility-report.json
```

## User Experience Considerations

### Progressive Enhancement
- **Core Functionality**: Essential features work without JavaScript
- **Enhanced Interaction**: JavaScript enhances but doesn't replace basic functionality
- **Graceful Degradation**: Fallbacks for advanced features
- **Performance**: Fast loading for assistive technology users

### Error Prevention and Recovery
```typescript
// Accessible error handling
const useFormValidation = () => {
  const [errors, setErrors] = useState<Record<string, string>>({});
  
  const validateField = (name: string, value: string) => {
    const error = validateValue(name, value);
    if (error) {
      setErrors(prev => ({ ...prev, [name]: error }));
      announceError(`${name}: ${error}`);
    } else {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors[name];
        return newErrors;
      });
    }
  };
  
  const announceError = (message: string) => {
    const announcement = document.getElementById('error-announcements');
    if (announcement) {
      announcement.textContent = message;
    }
  };
  
  return { errors, validateField };
};
```

### Documentation and Training
- **Accessibility Guidelines**: Internal accessibility standards and best practices
- **Component Documentation**: Accessibility features documented in Storybook
- **Training Materials**: Regular accessibility training for development team
- **User Testing**: Regular testing with users who have disabilities

## Compliance and Legal Requirements

### ADA Compliance
- **Title III Compliance**: Public accommodation requirements
- **Section 508**: Federal accessibility standards when applicable
- **State Regulations**: State-specific accessibility requirements
- **Documentation**: Accessibility conformance statements

### Accessibility Audit Process
- **Quarterly Audits**: Regular accessibility reviews
- **Third-Party Testing**: External accessibility audits
- **User Feedback**: Accessibility feedback collection and response
- **Remediation Planning**: Systematic approach to fixing accessibility issues

### Accessibility Metrics
- **Automated Testing**: % of components passing automated accessibility tests
- **Manual Testing**: Regular manual testing coverage
- **User Success Rates**: Task completion rates for users with disabilities
- **Support Requests**: Accessibility-related support ticket tracking
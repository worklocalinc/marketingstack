# Creative Builder

Create templates and variants for the Creative Generator service.

## Template Types
- **email** - Email templates with subject, body, CTAs
- **push** - Web push notification templates
- **lander** - Landing page templates
- **ad** - Ad creative templates

## Required Placeholders
Templates must include these placeholders for Messaging Core:
- `{{TRACKING_URL}}` - Full tracking URL with IDs
- `{{LANDER_URL}}` - Offer landing page
- `{{CONTACT_NAME}}` - Contact's name (if available)
- `{{UNSUBSCRIBE_URL}}` - Unsubscribe link

## Template Schema
```typescript
{
  creativeTemplateId: string,
  type: 'email' | 'push' | 'lander' | 'ad',
  name: string,
  verticalTags: string[],
  content: {
    subject?: string,      // email only
    body: string,
    cta?: string,
  },
  metadata: {
    createdAt: Date,
    updatedAt: Date,
  }
}
```

## Variant Schema
```typescript
{
  creativeVariantId: string,
  creativeTemplateId: string,  // parent template
  variantName: string,         // e.g., "A", "B", "control"
  changes: {
    subject?: string,
    body?: string,
    cta?: string,
  },
  performance: {
    sends: number,
    clicks: number,
    conversions: number,
  }
}
```

## Task
Create creative: $ARGUMENTS

Generate:
1. Template with proper placeholders
2. 2-3 A/B test variants
3. API call to register in Creative Generator
4. Update service documentation if new template type

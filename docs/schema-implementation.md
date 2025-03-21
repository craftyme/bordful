# Schema.org Job Posting Implementation

Bordful implements comprehensive schema.org structured data for job listings, enhancing SEO and enabling rich search results in Google and other search engines.

## Overview

The schema.org implementation follows the [JobPosting](https://schema.org/JobPosting) specification, providing detailed structured data for each job listing. This improves search engine visibility and enables features like Google Jobs rich results.

## Key Features

- **Complete Required Properties**: All essential properties (title, description, datePosted, hiringOrganization)
- **Rich Remote Job Support**: Proper jobLocationType and applicantLocationRequirements for telecommute positions
- **Comprehensive Salary Information**: Structured salary ranges with currency and time unit support
- **Visa & Eligibility Requirements**: Clear indication of visa sponsorship availability
- **Detailed Job Classifications**: Industry and occupational category support

## Airtable Integration

The schema implementation maps directly to fields in your Airtable base. The following fields are recommended:

### Required Fields
- `title` - Job title
- `description` - Job description
- `company` - Company name
- `posted_date` - Publication date

### Recommended Fields
- `valid_through` - Expiration date (defaults to 30 days from posted_date if not specified)
- `job_identifier` - Job reference code/ID
- `visa_sponsorship` - Whether visa sponsorship is available ("Yes"/"No"/"Not specified")

### Enhanced Schema Fields
- `skills` - Required skills for the position
- `qualifications` - Specific qualifications needed
- `education_requirements` - Educational background needed (formatted as EducationalOccupationalCredential)
- `experience_requirements` - Experience needed for the position (formatted as OccupationalExperienceRequirements)
- `responsibilities` - Key responsibilities of the role
- `industry` - Industry associated with the job position
- `occupational_category` - Job category (preferably using O*NET-SOC codes)

#### Schema Formatting Notes
- For `education_requirements`, text is intelligently parsed using a configurable mapping system:
  - Maps free-text input to standardized schema.org credential categories
  - Uses a keyword-based detection system for common degree types:
    - "Bachelor's degree..." → "BachelorDegree"
    - "Master's degree..." → "MasterDegree"
    - "PhD..." → "DoctoralDegree"
    - "Associate degree..." → "AssociateDegree"
    - "High School..." → "HighSchool"
    - "Certificate..." → "Certificate"
  - The mapping is easily extensible through the `EDUCATION_CREDENTIAL_MAP` (see [Configurable Education Credential Mapping](#configurable-education-credential-mapping))
  - Fully compliant with schema.org standards using the EducationalOccupationalCredential type
- For `experience_requirements`, text is intelligently parsed to extract months of experience:
  - Example: "3+ years experience" is converted to 36 months internally
  - Experience is formatted using the OccupationalExperienceRequirements schema type
- These schema enhancements ensure compliance with Google's Rich Results Test standards

## Technical Implementation

The schema is implemented using Next.js Script component to inject JSON-LD structured data into each job listing page. The implementation:

1. Supports all standard schema.org JobPosting properties
2. Properly formats data according to schema.org requirements
3. Conditionally includes properties based on available data
4. Provides fallbacks for missing but important information
5. Validates against Google's Rich Results Test requirements
6. Uses TypeScript type definitions from schema-dts for type safety

### schema-dts Integration

Bordful uses the [schema-dts](https://github.com/google/schema-dts) package to provide TypeScript type definitions for Schema.org structured data. This offers several advantages:

- **Type Safety**: Early detection of schema errors during development
- **Auto-completion**: IDE support for all schema.org properties and types
- **Validation**: Ensures the schema follows the correct structure
- **Maintainability**: Easier to update and extend with proper type checking

The job schema component imports the necessary types:

```typescript
import type {
  JobPosting,
  WithContext,
  MonetaryAmount,
  QuantitativeValue,
  Organization,
  Place,
  PostalAddress,
  Country,
} from "schema-dts";
```

### Configurable Education Credential Mapping

The implementation uses a flexible, configuration-based approach to map free-text education requirements to standardized schema.org credential categories. This design improves maintainability and makes future extensions easier:

```typescript
// Configuration map for education credential types and their keywords
const EDUCATION_CREDENTIAL_MAP: Record<string, string[]> = {
  "BachelorDegree": ["bachelor", "bs", "ba", "b.s.", "b.a."],
  "MasterDegree": ["master", "ms", "ma", "m.s.", "m.a.", "mba"],
  "DoctoralDegree": ["phd", "doctorate", "doctoral"],
  "AssociateDegree": ["associate", "aa", "a.a."],
  "HighSchool": ["high school", "secondary"],
  "Certificate": ["certificate", "certification"],
  "ProfessionalDegree": ["professional"],
};
```

This configuration-driven approach offers several advantages:

1. **Maintainability**: Adding new credential types or keywords is as simple as updating the map
2. **Separation of Concerns**: Credential mapping logic is separated from the parsing function
3. **Extensibility**: Easy to extend for international education systems or specialized credentials
4. **Readability**: The mapping between keywords and credential types is clearly visible
5. **Performance**: Efficient keyword matching using JavaScript's Array.some() method

The mapping is used by the `parseEducationCredential` function which scans the free-text education field for matching keywords and returns the appropriate schema.org credential type:

```typescript
function parseEducationCredential(education: string | null | undefined): string {
  if (!education) return "EducationalOccupationalCredential";
  
  const lowerEd = education.toLowerCase();
  
  // Check each credential type for matching keywords
  for (const [credentialType, keywords] of Object.entries(EDUCATION_CREDENTIAL_MAP)) {
    if (keywords.some(keyword => lowerEd.includes(keyword))) {
      return credentialType;
    }
  }
  
  // Default fallback value
  return "EducationalOccupationalCredential";
}
```

To extend this mapping for your specific needs, simply modify the `EDUCATION_CREDENTIAL_MAP` constant in `components/ui/job-schema.tsx`.

## SEO Benefits

- **Enhanced Visibility**: Jobs may appear in Google's job search experience
- **Rich Results**: Better display in search results with structured salary, location, and company information
- **Improved Click-Through Rate**: More informative search listings lead to higher CTR
- **Better User Matching**: Proper structured data helps search engines match jobs to relevant candidates

## Testing Your Schema

You can validate your job schema implementation using:

1. [Google's Rich Results Test](https://search.google.com/test/rich-results)
2. [Schema.org Validator](https://validator.schema.org/)
3. [Structured Data Testing Tool](https://www.schemaapp.com/tools/structured-data-testing-tool/)

## Example Schema

```json
{
  "@context": "https://schema.org/",
  "@type": "JobPosting",
  "title": "Senior Software Engineer",
  "description": "We're looking for a senior software engineer...",
  "datePosted": "2025-03-01T00:00:00.000Z",
  "validThrough": "2025-04-01T00:00:00.000Z",
  "hiringOrganization": {
    "@type": "Organization",
    "name": "Example Company"
  },
  "jobLocation": {
    "@type": "Place",
    "address": {
      "@type": "PostalAddress",
      "addressLocality": "San Francisco",
      "addressRegion": "CA",
      "addressCountry": "USA"
    }
  },
  "employmentType": "FULL_TIME",
  "baseSalary": {
    "@type": "MonetaryAmount",
    "currency": "USD",
    "value": {
      "@type": "QuantitativeValue",
      "minValue": 120000,
      "maxValue": 150000,
      "unitText": "YEAR"
    }
  },
  "skills": "React, TypeScript, Node.js",
  "qualifications": "Bachelor's degree in Computer Science or related field",
  "experienceRequirements": "5+ years of software development experience",
  "industry": "Software Development",
  "occupationalCategory": "15-1252.00 Software Developers",
  "jobBenefits": "Medical, dental, vision insurance, 401(k) matching, unlimited PTO",
  "directApply": false
}
``` 
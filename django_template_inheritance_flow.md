# Django Template Inheritance Flow

This diagram explains how Django template inheritance works, showing the relationship between base templates, child templates, and the rendering process.

## Template Inheritance Architecture

```mermaid
graph TB
    subgraph "Project Structure"
        A[myproject/] --> B[myapp/]
        B --> C[templates/]
        C --> D[myapp/]
        D --> E[base.html]
        D --> F[home.html]
        D --> G[about.html]
    end

    subgraph "Base Template (base.html)"
        H[<!DOCTYPE html>]
        I[{% block title %}My Website{% endblock %}]
        J[Header Section]
        K[Navigation]
        L[{% block content %}Default Content{% endblock %}]
        M[Footer]
        
        H --> I
        I --> J
        J --> K
        K --> L
        L --> M
    end

    subgraph "Child Templates"
        N[home.html<br/>{% extends 'myapp/base.html' %}]
        O[about.html<br/>{% extends 'myapp/base.html' %}]
        
        P[{% block title %}Home - My Website{% endblock %}]
        Q[{% block content %}Home Page Content{% endblock %}]
        
        R[{% block title %}About - My Website{% endblock %}]
        S[{% block content %}About Page Content{% endblock %}]
        
        N --> P
        N --> Q
        O --> R
        O --> S
    end

    E -.-> N
    E -.-> O
    
    style E fill:#e1f5fe
    style N fill:#f3e5f5
    style O fill:#f3e5f5
```

## Request-Response Flow

```mermaid
sequenceDiagram
    participant User
    participant Django
    participant URLConf
    participant View
    participant TemplateEngine
    participant BaseTemplate
    participant ChildTemplate

    User->>Django: GET /
    Django->>URLConf: Match URL pattern
    URLConf->>View: Call home(request)
    View->>TemplateEngine: render('myapp/home.html')
    
    TemplateEngine->>ChildTemplate: Load home.html
    ChildTemplate->>TemplateEngine: {% extends 'myapp/base.html' %}
    TemplateEngine->>BaseTemplate: Load base.html
    
    BaseTemplate->>TemplateEngine: Return base structure with blocks
    ChildTemplate->>TemplateEngine: Override specific blocks
    TemplateEngine->>TemplateEngine: Merge base + child content
    
    TemplateEngine->>View: Return rendered HTML
    View->>Django: Return HttpResponse
    Django->>User: Send complete HTML page
```

## Template Block Inheritance

```mermaid
graph LR
    subgraph "Base Template Blocks"
        A[{% block title %}]
        B[{% block content %}]
        C[Header - Fixed]
        D[Navigation - Fixed]
        E[Footer - Fixed]
    end

    subgraph "Home Template Override"
        F[{% block title %}<br/>Home - My Website]
        G[{% block content %}<br/>Welcome to Home Page]
    end

    subgraph "About Template Override"
        H[{% block title %}<br/>About - My Website]
        I[{% block content %}<br/>About Us Content]
    end

    subgraph "Final Rendered Pages"
        J[Home Page<br/>- Custom Title<br/>- Custom Content<br/>- Inherited Header/Nav/Footer]
        K[About Page<br/>- Custom Title<br/>- Custom Content<br/>- Inherited Header/Nav/Footer]
    end

    A -.-> F
    A -.-> H
    B -.-> G
    B -.-> I
    
    F --> J
    G --> J
    C --> J
    D --> J
    E --> J
    
    H --> K
    I --> K
    C --> K
    D --> K
    E --> K

    style A fill:#ffecb3
    style B fill:#ffecb3
    style F fill:#c8e6c9
    style G fill:#c8e6c9
    style H fill:#c8e6c9
    style I fill:#c8e6c9
```

## Key Benefits of Template Inheritance

```mermaid
mindmap
  root((Template Inheritance))
    Code Reusability
      Common HTML structure
      Shared CSS/JS
      Consistent layout
    Maintainability
      Single source of truth
      Easy global changes
      Reduced duplication
    Flexibility
      Override specific blocks
      Multiple inheritance levels
      Custom content per page
    Performance
      Template caching
      Efficient rendering
      Optimized loading
```

## Template Loading Process

```mermaid
flowchart TD
    A[Request arrives] --> B{URL matches pattern?}
    B -->|Yes| C[Call corresponding view]
    B -->|No| D[404 Error]
    
    C --> E[View calls render()]
    E --> F[Template engine starts]
    F --> G{Child template extends base?}
    
    G -->|Yes| H[Load base template]
    G -->|No| I[Render template directly]
    
    H --> J[Identify blocks in base]
    J --> K[Load child template blocks]
    K --> L[Merge base + child content]
    L --> M[Apply context variables]
    M --> N[Return rendered HTML]
    
    I --> M
    N --> O[Send response to user]
    
    style A fill:#e8f5e8
    style N fill:#e8f5e8
    style D fill:#ffebee
```

## Template Inheritance Hierarchy Example

```mermaid
graph TD
    A[base.html<br/>- DOCTYPE, head, body structure<br/>- Common CSS/JS<br/>- Header, Navigation, Footer<br/>- Block definitions] 
    
    B[section_base.html<br/>- Extends base.html<br/>- Section-specific layout<br/>- Additional blocks]
    
    C[home.html<br/>- Extends base.html<br/>- Home page content<br/>- Override title & content blocks]
    
    D[about.html<br/>- Extends base.html<br/>- About page content<br/>- Override title & content blocks]
    
    E[blog_base.html<br/>- Extends section_base.html<br/>- Blog-specific layout<br/>- Sidebar, breadcrumbs]
    
    F[blog_post.html<br/>- Extends blog_base.html<br/>- Individual post layout<br/>- Article content]
    
    A --> B
    A --> C
    A --> D
    B --> E
    E --> F
    
    style A fill:#1976d2,color:#fff
    style B fill:#388e3c,color:#fff
    style E fill:#388e3c,color:#fff
    style C fill:#f57c00,color:#fff
    style D fill:#f57c00,color:#fff
    style F fill:#7b1fa2,color:#fff
```

This diagram illustrates the complete flow of Django template inheritance, from the basic project structure through the rendering process, showing how base templates provide structure while child templates customize specific content blocks.
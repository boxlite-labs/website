# BoxLite SEO & GEO Checklist

This document outlines all SEO and GEO (Generative Engine Optimization) work done and ongoing tasks for the BoxLite website.

## Completed Items

### Technical SEO

- [x] **robots.txt** - Configured with correct sitemap URL
- [x] **Sitemap** - Auto-generated via @astrojs/sitemap
- [x] **Canonical URLs** - Set via astro-seo in Layout
- [x] **Meta descriptions** - All pages have unique descriptions
- [x] **JSON-LD Schema** - Structured data for Organization, Website, SoftwareApplication
- [x] **Breadcrumb schema** - Added to all content pages
- [x] **404 page** - SEO-optimized with helpful links
- [x] **Mobile responsive** - Tailwind CSS responsive design
- [x] **Google Search Console** - Verification placeholder added

### Content SEO

- [x] **FAQ page** - 14 FAQs with FAQPage schema
- [x] **Comparison pages** - BoxLite vs Docker/Firecracker/gVisor
- [x] **Use case pages** - 6 detailed use case pages
- [x] **Guide pages** - AI Agent Sandboxing comprehensive guide
- [x] **Blog posts** - 3 BoxLite-focused blog posts

### GEO (AI Citation Optimization)

- [x] **TL;DR sections** - All content pages start with summary
- [x] **Question-based headings** - "What is X?", "How does X work?"
- [x] **Short answer + deep dive format** - Structured for AI extraction
- [x] **Entity-first paragraphs** - "BoxLite is..." format
- [x] **Comparison content** - vs Docker, Firecracker, gVisor
- [x] **Technical credibility** - Code examples, architecture details

### Internal Linking

- [x] **Navigation** - Use Cases, Guides, Compare dropdowns
- [x] **Footer links** - Comprehensive footer with all sections
- [x] **Breadcrumbs** - Visual breadcrumbs on all content pages
- [x] **Cross-linking** - "Learn more" links between related pages

## Configuration Files

### Layout.astro Schema

The Layout component includes:
- Organization schema
- WebSite schema with SearchAction
- SoftwareApplication schema
- Dynamic breadcrumb schema

### astro.config.mjs Sitemap

```javascript
sitemap({
  filter: (page) => !page.includes('/404'),
  changefreq: 'weekly',
  priority: 0.7,
  lastmod: new Date()
})
```

## Pending Items

### Search Console Setup

1. Add your site to Google Search Console
2. Replace `XXXX` in Layout.astro with verification code:
   ```html
   <meta name="google-site-verification" content="your-verification-code" />
   ```
3. Submit sitemap: `https://boxlite-labs.github.io/website/sitemap-index.xml`

### Analytics Setup

1. Create Google Analytics 4 property
2. Replace placeholder in Layout.astro:
   ```html
   <script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXX"></script>
   ```

### Directory Submissions

Submit BoxLite to these directories:

**Developer Tools**
- [ ] Product Hunt - https://producthunt.com
- [ ] AlternativeTo - https://alternativeto.net
- [ ] StackShare - https://stackshare.io
- [ ] LibHunt - https://libhunt.com

**AI/LLM Directories**
- [ ] There's An AI For That - https://theresanaiforthat.com
- [ ] AI Tools Directory - https://aitoolsdirectory.com
- [ ] Futurepedia - https://futurepedia.io

**Developer Communities**
- [ ] Hacker News - https://news.ycombinator.com
- [ ] Reddit (r/devops, r/docker, r/python) - https://reddit.com
- [ ] Dev.to - https://dev.to
- [ ] Lobsters - https://lobste.rs

**Open Source**
- [ ] GitHub Topics - Add relevant topics to repo
- [ ] Awesome lists - Submit to relevant awesome-* lists

### Backlink Opportunities

- [ ] Guest posts on security/DevOps blogs
- [ ] HARO (Help A Reporter Out) responses
- [ ] Developer podcast appearances
- [ ] Conference talks and presentations
- [ ] Integration documentation on partner sites

### Content Calendar

Suggested ongoing content:

1. **Monthly blog posts** on:
   - Security best practices
   - Use case deep dives
   - Integration tutorials
   - Benchmark comparisons

2. **Technical documentation** on:
   - API reference
   - SDK guides
   - Architecture overview

3. **Case studies** when customers adopt BoxLite

## GEO Best Practices Summary

For AI citation optimization:

1. **Start with TL;DR** - First paragraph should be a complete summary
2. **Use question headings** - "What is BoxLite?", "How does it work?"
3. **Entity-first sentences** - "BoxLite is an embeddable VM runtime..."
4. **Provide concrete details** - Specifics like "~100ms startup"
5. **Include comparisons** - Clear vs Docker, vs Firecracker content
6. **Show authority** - Technical depth, code examples, architecture

## Monitoring

### Key Metrics to Track

- Google Search Console impressions and clicks
- Core Web Vitals scores
- Organic traffic growth
- Keyword rankings for target terms
- AI citation appearances (search for quotes in AI chatbots)

### Target Keywords

Primary:
- "AI agent sandboxing"
- "secure code execution"
- "micro-VM runtime"
- "embeddable VM"
- "Docker alternative security"

Secondary:
- "run untrusted code safely"
- "AI code execution sandbox"
- "hardware isolation Python"
- "Claude computer use sandbox"

## Resources

- [Google Search Console](https://search.google.com/search-console)
- [Google Rich Results Test](https://search.google.com/test/rich-results)
- [Schema.org Validator](https://validator.schema.org/)
- [PageSpeed Insights](https://pagespeed.web.dev/)

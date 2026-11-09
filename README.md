# Integration Modernization Portfolio

**Dr. Alexis S. Smith** | Integration and Data Platform Architect | Founder & Chief Executive Officer, [Ascend AI NOW, LLC](https://www.ascendainow.com)

This repository shows how I take over an integration estate that was built around one person and rebuild it into something the organization owns, documents and can run on its own. It covers the Microsoft stack I work in: Power Automate, Microsoft Dynamics 365 Business Central, Azure, Microsoft Fabric and Power BI, with Shopify and Amazon as commerce channels.

All organizations, systems and figures here are fictional or simulated. No client data or client deliverables are included.

---

## What is inside

| Folder | What it shows |
|---|---|
| [`demo/`](demo/) | **Interactive demonstration** of daily B2C order batching into Business Central: 1 sales order per channel per day, safe to rerun, with exceptions and daily validation. Open `demo/index.html` in any browser. |
| [`sample-assessment/`](sample-assessment/) | **Sample discovery deliverable** for a fictional company (Contoso Retail Group): executive summary, inventory, target architecture, order of work and a firm rebuild quote, plus the Excel inventory workbook behind it. |
| [`docs/architecture.md`](docs/architecture.md) | Architecture of the batching pattern, with a diagram and the design principles it follows. |
| [`docs/flow-logic.md`](docs/flow-logic.md) | Step-by-step flow logic with Power Automate expressions, rerun check and error handling. |
| [`docs/build-guide.md`](docs/build-guide.md) | How to build the pattern against a real Business Central sandbox and Shopify development store. |
| [`samples/`](samples/) | Business Central API v2.0 `salesOrders` request body for a consolidated daily order. |

## The approach

I work through 5 stages, from the Ascend AI NOW™ Framework: **Diagnose → Govern → Architect → Deploy → Scale.**

1. **Diagnose:** inventory every flow, connection, Azure resource and pipeline. Classify each as Keep, Merge, Retire or Owner Unknown, and find the connections running under departed staff or shared logins.
2. **Govern:** move every connection to a service identity the organization owns, with named environments, source control and change approval.
3. **Architect:** keep business logic in the systems that own the data, with thin, rerunnable flows and no database between enterprise systems.
4. **Deploy:** rebuild the integrations that matter, starting with order to cash, and run old and new in parallel until totals match.
5. **Scale:** document everything and validate every sync daily in Power BI, so the internal team runs and changes the estate without the consultant.

## Design principles demonstrated

- **Logic lives in the system that owns the data.** Pricing, tax and posting rules stay in Business Central.
- **Every run is rerunnable.** An external document number per channel and day prevents duplicate orders.
- **Tied to no individual.** Authentication through a Microsoft Entra app registration, with secrets in Azure Key Vault.
- **Exceptions are surfaced, not lost.** Unmapped items are held back and reported the same morning.
- **Portable.** Standard APIs and plain orchestration, so the logic can move platforms without a rewrite.

## About me

I bring 30 years of technology experience, 19 of them in director-level or higher roles, across regulated healthcare, municipal utilities, public safety and K-12 education. I led a 37-person enterprise and solution architecture organization at Blue Cross Blue Shield of North Carolina, served 5 years on its enterprise data governance team, and led about 10 system conversions through go-live at the National Information Solutions Cooperative (NISC).

- Website: [www.ascendainow.com](https://www.ascendainow.com)
- LinkedIn: [linkedin.com/in/alexisssmith](https://www.linkedin.com/in/alexisssmith)
- Email: alexis@ascendainow.com

---

© 2026 Ascend AI NOW, LLC. All rights reserved. Shared for portfolio review; not licensed for reuse without written permission.

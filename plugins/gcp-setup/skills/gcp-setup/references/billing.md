# GCP Billing Setup

## Info to Collect
- **Billing account**: Name or ID of existing billing account to link. If none exists, user needs to create one (MANUAL — involves payment details).
- **Budget alerts**: Whether to set up budget alerts, and at what thresholds (e.g., 50%, 90%, 100% of $X/month).
- **Budget amount**: Monthly budget in USD (or other currency).

## Console Steps

### Linking an Existing Billing Account
1. Navigate to `https://console.cloud.google.com/billing?project=PROJECT_ID`
2. If project has no billing: look for "Link a billing account" or "Enable billing"
3. take_snapshot to find the billing account dropdown
4. Select the correct billing account
5. **MANUAL STEP**: Ask user to confirm and click the "Set Account" or "Link" button — this authorizes charges
6. take_screenshot to verify billing is linked

### Creating Budget Alerts
1. Navigate to `https://console.cloud.google.com/billing/budgets?project=PROJECT_ID`
2. Click "Create Budget"
3. Fill budget name
4. Set scope (all projects or specific project)
5. Set budget amount
6. Configure alert thresholds (default: 50%, 90%, 100%)
7. Configure notification channels if needed
8. Click "Finish"

## Always Manual
- Creating a new billing account (requires payment method)
- Confirming billing account linkage
- Upgrading from free trial to paid account
- Modifying payment methods

## Common Issues
- **No billing accounts visible**: User may not have Billing Account Administrator role
- **Organization policy**: Some orgs restrict billing account linking
- **Free trial**: New accounts are on free trial with $300 credit — some APIs behave differently

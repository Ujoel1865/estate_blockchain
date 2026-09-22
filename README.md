# SecureRealEstatePlatform

A Solidity smart contract for **fractional real-estate investment using USDT**.

The platform allows real-estate companies to register properties, divide properties into digital shares, and allow investors to purchase those shares using USDT. The contract keeps track of investor ownership, collects platform fees, manages property funds, and distributes dividends to shareholders on-chain.

The current contract is designed for **TRON-compatible USDT environments** and uses Solidity `0.5.10`.

---

## Overview

Traditional real-estate investment often requires investors to provide a large amount of capital to participate in a property.

SecureRealEstatePlatform is designed to make this process more accessible by dividing a property into a fixed number of digital shares.

For example:

```text
Property
   │
   ├── 1,000 Shares
   │
   ├── Share Price: 100 USDT
   │
   └── Total Value: 100,000 USDT
             │
             ├── Investor A → 100 shares
             ├── Investor B → 250 shares
             └── Investor C → 50 shares


Each investor's share ownership is recorded on-chain.

The platform also supports:

Real-estate company registration
Property creation
Fractional property shares
USDT share purchases
Platform fees
Property fund management
Fund liquidation
Dividend deposits
On-chain dividend accounting
Investor dividend claims
Role-based access control
Reentrancy protection
Legacy USDT compatibility
Main Features
1. Company Registration

A real-estate company can be registered with:

Company name
Company wallet
Automatically generated company code
Optional company image

Example:

Company Name: Luxury Homes Inc
Generated Code: LUX001
Wallet: TRON address

The contract stores the company and associates its wallet with the company ID.

Function
registerCompany(
    string memory companyName,
    address companyWallet,
    string memory image
)
2. Property Creation

A registered company can create properties.

Each property contains:

Property ID
Company ID
Property code
Property URI
Total number of shares
Price per share
Number of shares sold
Total amount raised
Property status
Fundraising wallet
Dividend wallet
Property image

Example:

Property: Luxury Apartment Project

Total Shares: 1,000
Price Per Share: 100 USDT

Total Property Funding:
1,000 × 100 USDT = 100,000 USDT
Function
createProperty(
    address companyWallet,
    string memory propertyURI,
    uint256 totalShares,
    uint256 sharePrice,
    address fundraisingWallet,
    address dividendWallet,
    string memory image
)

The contract automatically generates a property code based on the company's code.

Example:

LUX001
LUX001-P1
LUX001-P2
3. Buying Property Shares

Investors purchase property shares using USDT.

The investor must first approve the smart contract to spend the required amount of USDT.

The total amount charged is:

Share Cost + Platform Buy Fee

For example:

Share price:       100 USDT
Shares purchased:  5

Share cost:
100 × 5 = 500 USDT

Buy fee:
4.10 USDT

Total:
504.10 USDT

The share cost is retained by the contract as property funds.

The platform fee is transferred to the developer wallet.

Function
buyShares(
    uint256 propertyId,
    uint256 shareCount
)

The contract records the investor's ownership:

investorShares[investor][propertyId]
4. Property Funding

When investors purchase shares, the contract tracks the money raised for that property.

The contract maintains:

propertyFunds[propertyId]

This represents funds held by the contract for the property.

The amount raised by the property is also tracked through:

properties[propertyId].totalRaised

The number of shares sold is tracked through:

properties[propertyId].sharesSold
5. Property Status

Each property can have one of four statuses:

enum PropertyStatus {
    ACTIVE,
    FUNDED,
    COMPLETED,
    CANCELLED
}
ACTIVE

The property is accepting investments.

FUNDED

All available shares have been sold.

COMPLETED

The company owner has manually closed the sales.

CANCELLED

Reserved as a possible future state for cancellation logic.

Note: The current contract defines CANCELLED, but does not currently implement a public function that changes a property to CANCELLED.

6. Closing Property Sales

The company owner can close sales for an active property.

Function
closeSales(uint256 propertyId)

Only the wallet registered as the company's wallet can perform this operation.

The property status changes from:

ACTIVE
   ↓
COMPLETED
7. Fund Liquidation

Once a property is funded or completed, the company owner can withdraw property funds from the contract.

A liquidation fee is deducted.

The default liquidation fee is:

10%

Example:

Amount liquidated: 50,000 USDT

10% developer fee:
5,000 USDT

Company receives:
45,000 USDT

The developer receives the liquidation fee and the company's registered wallet receives the remaining amount.

Function
liquidateFunds(
    uint256 propertyId,
    uint256 amount
)

The contract updates the property's available funds before transferring the tokens.

This helps reduce the risk associated with reentrant calls.

8. Dividend Distribution

The platform supports on-chain dividend accounting.

A property's designated dividend wallet can deposit USDT into the contract.

The platform administrator can also deposit dividends.

Function
depositDividends(
    uint256 propertyId,
    uint256 amount
)

The deposited amount is divided across the property's currently sold shares.

The contract uses a cumulative dividend-per-share model.

9. Dividend Calculation

The contract uses:

POINTS = 10**18

to maintain precision when calculating fractional dividend amounts.

The accumulated dividend per share is stored in:

accDividendPerShare[propertyId]

For example:

Dividend deposited: 10,000 USDT
Shares sold:        1,000

Dividend per share:
10 USDT

An investor holding:

100 shares

would have:

100 × 10 USDT
= 1,000 USDT

in dividends before accounting for previously credited amounts.

10. Claiming Dividends

Investors can claim dividends they are entitled to.

Function
claimDividends(uint256 propertyId)

The contract calculates:

Investor Shares
        ×
Accumulated Dividend Per Share
        -
Previously Credited Dividend

Only the unclaimed amount is transferred to the investor.

The contract then updates the investor's dividend accounting before transferring the USDT.

11. USDT Handling

The contract uses an IERC20 interface to interact with the configured USDT token.

The contract constructor receives the USDT token address:

constructor(
    address _usdtToken,
    address _developerWallet
)

The USDT address is stored in:

usdtToken

The developer fee wallet is stored in:

developerWallet
12. Legacy USDT Compatibility

Some older USDT implementations do not return a boolean value from:

transfer()
transferFrom()

Because of this, the contract does not rely on the standard ERC-20 boolean return value.

Instead, SafeERC20 performs a low-level call:

(bool success, ) = address(token).call(...)

and verifies that the call itself succeeded.

This is intended to improve compatibility with legacy USDT implementations.

13. Reentrancy Protection

Functions that move funds use the nonReentrant modifier.

The contract implements its own lightweight ReentrancyGuard.

Protected functions include:

buyShares()
liquidateFunds()
depositDividends()
claimDividends()

The guard prevents a contract from repeatedly entering a protected function before the previous execution has completed.

14. Access Control

There are two main permission levels.

Platform Administrator

The administrator is the address that deploys the contract.

The administrator can:

Transfer admin ownership
Change platform fees
Recover non-USDT tokens

Functions:

transferAdmin()
updateFees()
recoverTokens()
Company Owner

The registered company wallet controls its properties.

A company owner can:

Close property sales
Liquidate property funds

The contract verifies ownership using:

companies[companyId].wallet
15. Platform Fees

The default share purchase fee is:

4.10 USDT

Internally:

buyFee = 4100000;

because USDT commonly uses 6 decimal places.

The default liquidation fee is:

10%

represented as:

liquidationFeePercent = 1000;

The contract uses basis points for liquidation fees:

1000 / 10000 = 10%

The administrator can update these fees.

The maximum liquidation fee allowed by the current contract is:

20%
16. Data Structure

The contract uses two main structs.

Company
struct Company {
    uint256 companyId;
    string companyCode;
    string name;
    address wallet;
    bool registered;
    uint256 propertyCount;
    string image;
}
Property
struct Property {
    uint256 propertyId;
    uint256 companyId;
    string propertyCode;
    string propertyURI;
    uint256 totalShares;
    uint256 sharePrice;
    uint256 totalRaised;
    uint256 sharesSold;
    PropertyStatus status;
    address fundraisingWallet;
    address dividendWallet;
    string image;
}
17. Important Mappings

The contract uses mappings to efficiently track relationships between companies, properties and investors.

Companies
mapping(uint256 => Company) public companies;

Stores companies by ID.

Properties
mapping(uint256 => Property) public properties;

Stores properties by ID.

Company Codes
mapping(string => uint256) public codeToCompanyId;

Allows a company code to be mapped to a company ID.

Property Codes
mapping(string => uint256) public codeToPropertyId;

Allows a property code to be mapped to a property ID.

Company Wallets
mapping(address => uint256) public walletToCompanyId;

Maps a company wallet to its registered company.

Company Properties
mapping(uint256 => uint256[]) public companyProperties;

Stores all properties belonging to a company.

Investor Ownership
mapping(address => mapping(uint256 => uint256)) public investorShares;

Tracks how many shares an investor owns in each property.

Property Funds
mapping(uint256 => uint256) public propertyFunds;

Tracks funds held by the contract for each property.

18. Events

The contract emits events for important actions.

Company Registered
CompanyRegistered
Property Created
PropertyCreated
Shares Purchased
SharesPurchased
Funds Liquidated
FundsLiquidated
Sales Closed
SalesClosed
Dividends Deposited
DividendsDeposited
Dividends Claimed
DividendsClaimed

These events allow a frontend, explorer or indexing service to track activity without repeatedly reading the entire contract state.

19. Basic Transaction Flow

The intended platform flow is:

                  ┌──────────────────────┐
                  │  Real Estate Company │
                  └──────────┬───────────┘
                             │
                             ▼
                    Register Company
                             │
                             ▼
                      Create Property
                             │
                             ▼
                    Property ACTIVE
                             │
                             ▼
                  ┌──────────────────────┐
                  │       Investor       │
                  └──────────┬───────────┘
                             │
                        Approve USDT
                             │
                             ▼
                        Buy Shares
                             │
                             ▼
                  USDT → Smart Contract
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
       Property Funds                 Developer Fee
              │
              ▼
       Property Funded
              │
              ▼
       Company Liquidates
              │
       ┌──────┴──────┐
       │             │
       ▼             ▼
 Developer Fee   Company Funds
       
       
Dividend Flow:

Dividend Wallet
       │
       ▼
Deposit USDT
       │
       ▼
Smart Contract
       │
       ▼
Dividend Per Share
       │
       ▼
Investor Claims
       │
       ▼
Investor USDT
20. Example Investment Flow

Suppose a property has:

Total Shares:     10,000
Price Per Share:  10 USDT

An investor purchases:

500 shares

The investment amount is:

500 × 10
= 5,000 USDT

The investor also pays the platform's purchase fee:

4.10 USDT

Total USDT transferred from the investor:

5,004.10 USDT

The contract records:

Investor ownership = 500 shares

Property raised = +5,000 USDT

Developer fee = +4.10 USDT

The investor's ownership is permanently recorded on-chain until the contract's business logic changes it.

21. Technology

The contract uses:

Solidity 0.5.10
ERC-20 compatible token interface
SafeMath
Custom SafeERC20 compatibility layer
ReentrancyGuard
Solidity mappings and structs
Event-based transaction tracking
22. Repository Structure

A typical repository can be organized as:

SecureRealEstatePlatform/
│
├── contracts/
│   └── SecureRealEstatePlatform.sol
│
├── test/
│   └── ...
│
├── scripts/
│   └── ...
│
├── frontend/
│   └── ...
│
└── README.md

The smart contract contains the core financial and ownership logic.

The frontend should interact with the contract rather than maintaining its own independent source of truth for:

Property ownership
Shares sold
Property funds
Dividend accounting
Company registration
23. Deployment

The constructor requires two addresses:

constructor(
    address _usdtToken,
    address _developerWallet
)

Example:

USDT Token Address
        +
Developer Wallet
        ↓
SecureRealEstatePlatform

The deploying wallet automatically becomes:

platformAdmin
Before deployment

Verify:

The correct USDT contract address is being used.
The developer wallet is correct.
The contract is compiled with the intended Solidity version.
The deployment network is correct.
The contract has been tested on a testnet.
The contract has undergone an independent security review before handling real funds.
24. Testing

The project should first be tested on a testnet before any mainnet deployment.

Recommended test cases include:

Company
Register a company
Prevent duplicate company wallets
Reject empty company names
Reject invalid wallets
Property
Create a property
Reject unregistered companies
Reject zero shares
Reject zero share prices
Prevent duplicate property codes
Investment
Approve USDT
Buy shares
Prevent purchases above available shares
Prevent purchases without sufficient USDT
Verify investor ownership
Verify platform fee
Verify property funds
Funding
Verify property becomes FUNDED when all shares are sold
Close active sales
Prevent unauthorized company actions
Liquidation
Liquidate property funds
Verify developer fee
Verify company proceeds
Prevent liquidation above available funds
Prevent unauthorized liquidation
Dividends
Deposit dividends
Calculate dividend per share
Claim dividends
Prevent claiming without shares
Prevent claiming the same dividend twice
Verify contract balance after claims
25. Security Considerations

This contract includes several protections, including:

SafeMath arithmetic checks
Reentrancy protection
Access control
Zero-address validation
Share availability checks
USDT balance checks
USDT allowance checks
State updates before token transfers in fund liquidation
Protection against recovering the configured USDT token

However, these protections do not mean the contract is automatically production-ready or fully audited.

Before mainnet deployment and real-money use, the contract should undergo:

Independent smart-contract audit
Full unit testing
Integration testing
Economic/business-logic review
Token compatibility testing
Access-control review
Reentrancy review
Dividend-accounting review
Legal and regulatory review for the target jurisdictions
26. Current Limitations

The current version is an MVP-level smart contract and has areas that should be addressed before production.

Property cancellation

CANCELLED exists as a status but there is currently no cancellation function.

Property verification

The contract stores property information but does not independently verify that a real-world property exists.

Legal ownership

Owning a blockchain share does not automatically establish legal ownership of a physical property. Legal structures must connect the blockchain representation to the underlying real-world asset.

Investor identity

The current contract does not implement KYC/AML.

Regulatory compliance

Real-estate securities and fractional ownership may be regulated differently depending on the jurisdiction.

Token standard assumptions

The contract expects the configured token to behave like the supported USDT interface. The actual token implementation must be tested before deployment.

Upgradeability

The contract is not upgradeable. Once deployed, its code cannot simply be replaced.

27. Roadmap
Phase 1 — Smart Contract
 Company registration
 Property creation
 Fractional shares
 USDT share purchases
 Platform fees
 Property fund accounting
 Fund liquidation
 Dividend deposits
 Dividend claims
 Reentrancy protection
 Full automated test suite
 Independent security audit
Phase 2 — Platform
 Investor dashboard
 Real-estate company dashboard
 Property listing
 Wallet connection
 Investment interface
 Dividend dashboard
 Transaction history
 Admin dashboard
Phase 3 — Real-World Asset Integration
 Property verification
 Company verification
 KYC/AML integration
 Legal ownership structure
 First real-world property
 First real-estate company
 Pilot investors
Phase 4 — Mainnet
 Final security audit
 Mainnet deployment
 Production frontend
 Monitoring and analytics
 Investor onboarding
 Real-world property onboarding
28. Disclaimer

This repository contains experimental blockchain software.

The smart contract is not financial advice and should not be used to handle real investor funds without appropriate technical, legal and regulatory review.

Tokenizing a real-world property does not by itself create legal ownership of that property. A compliant legal structure is required to connect the blockchain asset to the underlying real estate.

License

This project is released under the GNU General Public License v3.0 (GPL-3.0).


One thing I deliberately changed from your Solidity comment: I **wouldn't put “Mainnet compatible with proper security measures” in the README as a claim**. Your contract has a solid foundation, but because it handles investor money, an audit and legal structure are still needed before calling it production-ready.

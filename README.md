
# Vista Service Daily Network Availability Exclusion System

## 📋 Executive Summary

**System Purpose**: A production-grade web application for telecommunications network performance monitoring that calculates hypothetical network availability by excluding specific outage categories, enabling root cause analysis and performance improvement planning.

**Core Function**: Transform raw incident ticket data into actionable insights by calculating network availability exclusions based on categorized outages across network technologies (2G, 3G, 4G, TDD, 5G).

**Primary Users**: Network Operations Teams, Management, Network Planners

---

## 🎯 Key Objectives & Business Value

### **Primary Objectives**
1. **Root Cause Analysis**: Identify specific outage categories impacting network availability
2. **Performance Benchmarking**: Calculate potential availability improvements by addressing specific issues
3. **Historical Trend Analysis**: Track availability performance over time across all technologies
4. **Data-Driven Decision Making**: Provide quantitative basis for resource allocation and improvement initiatives

### **Target Metrics**
- **Universal Availability Target**: 98.5% for all technologies
- **Measurement Frequency**: Daily calculations
- **Historical Depth**: 10-day trending window
- **Calculation Accuracy**: Two decimal precision

---

## 🏗️ System Architecture

### **Technology Stack**
| Component | Technology | Version |
|-----------|------------|---------|
| **Frontend** | HTML5, CSS3, JavaScript (ES6+) | - |
| **Data Visualization** | Highcharts.js | v10+ |
| **Backend Processing** | Python 3.8+ with Pandas | Latest stable |
| **Data Storage** | PostgreSQL (Production) / File-based (Prototype) | - |
| **Deployment** | Single HTML (Prototype) / Full-stack (Production) | - |

### **Architectural Diagram**
```
┌─────────────────────────────────────────────────────────────────────┐
│                          User Interface Layer                       │
├─────────────────────────────────────────────────────────────────────┤
│  Authentication → Dashboard → Data Upload → Results → Export Tools   │
└───────────────────────────────┬─────────────────────────────────────┘
                                │ HTTP/REST
┌───────────────────────────────▼─────────────────────────────────────┐
│                         Application Layer                           │
├─────────────────────────────────────────────────────────────────────┤
│     User Management    │    ETL Controller    │   Calculation Engine│
└───────────────────────────────┬─────────────────────────────────────┘
                                │
┌───────────────────────────────▼─────────────────────────────────────┐
│                          Data Processing Layer                      │
├─────────────────────────────────────────────────────────────────────┤
│  Python ETL Pipeline with Pandas for ticket processing & calculation│
└───────────────────────────────┬─────────────────────────────────────┘
                                │
┌───────────────────────────────▼─────────────────────────────────────┐
│                           Storage Layer                             │
├─────────────────────────────────────────────────────────────────────┤
│  PostgreSQL (Production) / Excel/CSV Files (Prototype)              │
│  • processed_data table          • cleaned_tickets table            │
│  • availability_tracker table    • SubRootcuz lookup file           │
└─────────────────────────────────────────────────────────────────────┘
```

### **Component Interaction Flow**
1. **User Authentication** → Domain validation (@iranvs.ir)
2. **Data Upload** → Ticket Excel + Availability inputs
3. **ETL Processing** → Python pipeline execution
4. **Calculation Engine** → Exclusion formulas applied
5. **Storage Update** → Overwrite existing date data
6. **Visualization** → Charts and tables rendered
7. **Export** → Results downloadable in multiple formats

---

## 🔄 Data Processing Pipeline (ETL)

### **Input Specifications**

#### **1. Ticket Data File (Required)**
**Format**: Excel (.xlsx/.xls)
**Required Columns** (case-insensitive):
- `Ticket ID` (string)
- `Fault First Occur Time` (datetime)
- `Fault Recovery Time(Process TT_faultrecoverytime)` (datetime)
- `Root Cause(Process TT_root_cause)` (string)
- `Probable Cause(Process TT_possiblecause)` (string)
- `Sub Root Cause(Process TT_sub_root_cause)` (string)
- `Probable Sub Cause(Process TT_probsubcause)` (string)
- `Site Province(Create TT_province)` (string)
- `2G-Affected Site Count(Create TT_sitecount2g)` (numeric)
- `3G-Affected Site Count(Create TT_sitecount3g)` (numeric)
- `4G-Affected Site Count(Create TT_sitecount4g)` (numeric)
- `TDD Affected Site Count(Create TT_tddsitecount)` (numeric)
- `5G-Affected Site Count(Create TT_sitecount5g)` (numeric)

#### **2. Lookup Data File (Optional)**
**Default File**: `SubRootcuz.xlsx`
**Purpose**: Maps root/sub causes to categories
**Columns Required**:
- `RC`: Root Cause (matches Aggregated_RC)
- `SC`: Sub Cause (matches Aggregated_SC)
- `CATEGORY`: One of 7 base categories
- `MS/NON MS`: MS classification

#### **3. Availability Inputs**
**Format**: Numerical values (0-100) with two decimal places
**Technologies**: 2G, 3G, 4G, TDD, 5G
**Default Value**: 98.5 for all technologies

### **ETL Processing Steps (Python Implementation)**

#### **Step 1: Data Loading & Initial Cleaning**
```python
def load_and_clean_tickets(ticket_file, target_date):
    """
    Load ticket data and apply initial cleaning rules
    """
    # Load Excel file
    df = pd.read_excel(ticket_file)
    
    # Convert date columns to datetime
    df['Fault First Occur Time'] = pd.to_datetime(df['Fault First Occur Time'])
    df['Fault Recovery Time'] = pd.to_datetime(df['Fault Recovery Time'])
    
    return df
```

#### **Step 2: Temporal Boundary Adjustment**
```python
def adjust_temporal_boundaries(df, target_date):
    """
    Adjust fault times to fit within target date boundaries
    """
    target_start = target_date.replace(hour=0, minute=0, second=0)
    target_end = target_date.replace(hour=23, minute=59, second=59)
    
    # Adjust Fault First Occur Time
    df['Adjusted_Fault_Time'] = df['Fault First Occur Time'].apply(
        lambda x: target_start if x < target_start else x
    )
    
    # Adjust Fault Recovery Time
    df['Adjusted_Recovery_Time'] = df['Fault Recovery Time'].apply(
        lambda x: target_end if pd.isna(x) or x > target_end else x
    )
    
    return df
```

#### **Step 3: MTTR Calculation**
```python
def calculate_mttr(df):
    """
    Calculate MTTR in seconds, capped at 24 hours
    """
    # Calculate duration in seconds
    df['MTTR_sec'] = (df['Adjusted_Recovery_Time'] - df['Adjusted_Fault_Time']).dt.total_seconds()
    
    # Cap at 23:59:59 (86399 seconds)
    df['MTTR_sec'] = df['MTTR_sec'].clip(upper=86399)
    
    # Convert to HH:MM:SS format
    df['MTTR'] = pd.to_timedelta(df['MTTR_sec'], unit='s').astype(str)
    
    return df
```

#### **Step 4: Cause Aggregation**
```python
def aggregate_causes(df):
    """
    Aggregate root and sub causes with priority logic
    """
    # Root Cause Aggregation (Priority: Root Cause > Probable Cause)
    df['Aggregated_RC'] = np.where(
        df['Root Cause(Process TT_root_cause)'].notna(),
        df['Root Cause(Process TT_root_cause)'],
        df['Probable Cause(Process TT_possiblecause)']
    )
    df['Aggregated_RC'] = df['Aggregated_RC'].fillna("BSS - Under Investigation")
    
    # Sub Cause Aggregation (Priority: Sub Root Cause > Probable Sub Cause)
    df['Aggregated_SC'] = np.where(
        df['Sub Root Cause(Process TT_sub_root_cause)'].notna(),
        df['Sub Root Cause(Process TT_sub_root_cause)'],
        df['Probable Sub Cause(Process TT_probsubcause)']
    )
    df['Aggregated_SC'] = df['Aggregated_SC'].fillna("Unclear")
    
    return df
```

#### **Step 5: Data Filtering**
```python
def filter_invalid_records(df):
    """
    Remove invalid records based on business rules
    """
    # Filter out rows where Aggregated_SC is exactly "ER" (case-sensitive, stripped)
    df = df[df['Aggregated_SC'].str.strip() != "ER"]
    
    # Filter out rows with empty Site Province
    df = df[df['Site Province(Create TT_province)'].notna()]
    df = df[df['Site Province(Create TT_province)'].str.strip() != ""]
    
    return df
```

#### **Step 6: Category Mapping**
```python
def map_categories(df, lookup_file):
    """
    Map aggregated causes to categories using lookup file
    """
    # Load lookup file
    lookup_df = pd.read_excel(lookup_file)
    
    # Merge with ticket data
    df = pd.merge(
        df,
        lookup_df,
        left_on=['Aggregated_RC', 'Aggregated_SC'],
        right_on=['RC', 'SC'],
        how='left'
    )
    
    # Fill missing categories
    df['CATEGORY'] = df['CATEGORY'].fillna("Others")
    
    return df
```

#### **Step 7: Outage Calculation**
```python
def calculate_technology_outages(df):
    """
    Calculate outage seconds per technology
    """
    technologies = ['2G', '3G', '4G', 'TDD', '5G']
    site_count_columns = {
        '2G': '2G-Affected Site Count(Create TT_sitecount2g)',
        '3G': '3G-Affected Site Count(Create TT_sitecount3g)',
        '4G': '4G-Affected Site Count(Create TT_sitecount4g)',
        'TDD': 'TDD Affected Site Count(Create TT_tddsitecount)',
        '5G': '5G-Affected Site Count(Create TT_sitecount5g)'
    }
    
    for tech in technologies:
        df[f'{tech}_outage_sec'] = df['MTTR_sec'] * df[site_count_columns[tech]]
    
    return df
```

---

## 🧮 Calculation Methodology

### **Base Categories**
The system uses 7 base categories for outage classification:
1. **Power** - Power supply issues
2. **NOA** - Network Operating Agreement related
3. **Non Telecom** - Non-telecommunications infrastructure issues
4. **PA / UA** - Planned Activity / Unplanned Activity
5. **Spare** - Spare parts unavailability
6. **Third Party** - Third-party service provider issues
7. **Others** - Unclassified or miscellaneous issues

### **Outage Calculation Formula**
```
Outage_Tech_Category = Σ(MTTR_sec_i × Affected_Sites_Tech_i)
```
Where:
- `i` represents each ticket in the category for the selected date
- `MTTR_sec_i` is the MTTR in seconds for ticket i
- `Affected_Sites_Tech_i` is the affected site count for the technology

### **Exclusion Calculation Formula**
```
Exclusion_Tech_Category = Avail_Tech + (Outage_Tech_Category × (100 - Avail_Tech) / Total_Outage_Tech)
```

#### **Formula Components:**
- **Avail_Tech**: Input availability percentage for the technology (e.g., 98.56)
- **Outage_Tech_Category**: Sum of outage seconds for the specific category
- **Total_Outage_Tech**: Sum of all outage seconds for the technology across all categories

#### **Calculation Rules:**
1. **Rounding**: Results rounded to 2 decimal places
2. **Zero Outage Handling**: If Total_Outage_Tech = 0, Exclusion = Avail_Tech
3. **Value Range**: Calculated exclusion can exceed 100% (capped at 100% for display)
4. **Missing Data**: Dates with no data are excluded from trend calculations

### **Example Calculation**
**Given:**
- 2G Availability = 98.56%
- Power outage for 2G = 150,000 site-seconds
- Total 2G outage = 1,000,000 site-seconds

**Calculation:**
```
Exclusion_2G_Power = 98.56 + (150,000 × (100 - 98.56) / 1,000,000)
                   = 98.56 + (150,000 × 1.44 / 1,000,000)
                   = 98.56 + 0.216
                   = 98.78%
```

### **Category Combinations**
The system generates all possible non-empty subsets of the 7 base categories:
- **Single categories**: "Power", "NOA", etc. (7 combinations)
- **Pair combinations**: "Power and NOA", "Power and Non Telecom", etc. (21 combinations)
- **Triple combinations**: "Power and NOA and Non Telecom", etc. (35 combinations)
- **All categories combined**: Single row representing exclusion of all outages

**Total Combinations**: 2^7 - 1 = 127 possible category combinations

### **Actual Availability Formula**
For reference, the actual availability calculation:
```
Actual_Availability_Tech = 100 - (Total_Outage_Seconds_Tech / (Total_Sites_Tech × 86400)) × 100
```
Where 86400 represents seconds in a day.

---

## 💾 Data Storage Architecture

### **Production Database Schema (PostgreSQL)**

#### **Table: processed_data**
| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | Unique identifier |
| date | DATE | NOT NULL | Target date (YYYY-MM-DD) |
| technology | VARCHAR(10) | NOT NULL | 2G/3G/4G/TDD/5G |
| category | VARCHAR(100) | NOT NULL | Category name |
| outage_seconds | BIGINT | NOT NULL | Total outage in seconds |
| site_count | INTEGER | NOT NULL | Number of affected sites |
| base_availability | DECIMAL(5,2) | NOT NULL | Input availability |
| exclusion_value | DECIMAL(5,2) | NOT NULL | Calculated exclusion |
| created_at | TIMESTAMP | DEFAULT NOW() | Processing timestamp |
| updated_at | TIMESTAMP | DEFAULT NOW() | Last update timestamp |

**Indexes:**
- `idx_processed_data_date_tech` ON processed_data(date, technology)
- `idx_processed_data_category` ON processed_data(category)

#### **Table: cleaned_tickets**
| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| date | DATE | Target date |
| ticket_id | VARCHAR(100) | Original ticket ID |
| technology | VARCHAR(10) | Technology affected |
| category | VARCHAR(100) | Mapped category |
| mttr_sec | INTEGER | MTTR in seconds |
| site_count | INTEGER | Affected site count |
| root_cause | VARCHAR(255) | Aggregated root cause |
| sub_cause | VARCHAR(255) | Aggregated sub cause |
| province | VARCHAR(100) | Site province |
| raw_data | JSONB | Original ticket data |

#### **Table: availability_tracker**
**Format**: Excel file with separate sheets per technology
**Structure**:
```
Sheet: 2G_outage
+---------------------+------------+------------+-----+------------+
| Metric              | 2024-01-15 | 2024-01-16 | ... | 2024-01-24 |
+---------------------+------------+------------+-----+------------+
| 2G_TCHA             | 98.56      | 98.50      | ... | 98.60      |
| 2G_outage_Power_ex  | 98.78      | 98.72      | ... | 98.81      |
| 2G_outage_NOA_ex    | 98.65      | 98.59      | ... | 98.68      |
| ...                 | ...        | ...        | ... | ...        |
+---------------------+------------+------------+-----+------------+
```

### **Prototype Storage Structure (File-based)**
```javascript
MOCK_DB = {
  processedData: {
    "2024-01-15": {
      tcha: { "2G": 98.56, "3G": 96.56, "4G": 98.05, "TDD": 90.00, "5G": 98.16 },
      rows: [
        {
          category: "Power",
          minutes: { "2G": 2500, "3G": 1800, "4G": 3200, "TDD": 1500, "5G": 2100 },
          sites: { "2G": 45, "3G": 32, "4G": 68, "TDD": 25, "5G": 38 }
        }
      ],
      techData: {
        "2G": {
          totalOutage: 15000,
          totalSites: 250,
          chartData: [...],
          outageDistribution: { Power: 2500, NOA: 1800, ... }
        }
      },
      customRows: [
        {
          category: "Infrastructure Issues",
          minutes: { "2G": 4200, ... },
          combinedCategories: ["Power", "PA / UA", "Spare"]
        }
      ]
    }
  }
}
```

### **Data Update & Overwrite Logic**
```python
def update_data_storage(target_date, new_data, storage_type='production'):
    """
    Update storage with new data for a specific date
    """
    if storage_type == 'production':
        # PostgreSQL update logic
        # 1. Delete existing records for target_date
        # 2. Insert new records
        # 3. Update tracker file
    else:
        # File-based update logic
        # 1. Check if date exists in MOCK_DB
        # 2. Overwrite if exists, create if new
        # 3. Update outage_tracker.xlsx
        
    return {
        "status": "success",
        "action": "overwritten" if date_exists else "created",
        "records_processed": len(new_data)
    }
```

**Overwrite Rule**: If data exists for selected date, completely replace it (no versioning)

---

## 🎨 User Interface Specifications

### **Terminology Standardization**
- **Replace "TCHA" with "Avail."** throughout UI
- **Consistent Naming**:
  - Input labels: "Target Avail. 2G"
  - Table headers: "Base Avail."
  - Chart labels: "Avail. Target (98.5%)"

### **Color Scheme & Styling**
| Element | Color | Hex Code | Usage |
|---------|-------|----------|-------|
| Primary Brand | Vista Blue | #2549A0 | Headers, primary buttons |
| Secondary | Vista Green | #78DD12 | Success states, actions |
| Target Line | Blue (Dashed) | #2549A0 | 98.5% reference line |
| Custom Rows | Light Green | #E8F5E9 | Background for user-created rows |
| Category Colors | Predefined Set | - | Consistent across all charts |

### **Layout Grid System**
- **Main Container**: Max width 1200px, centered
- **Chart Grid**: Responsive (1-3 columns based on screen size)
- **Form Layout**: Single column on mobile, multi-column on desktop
- **Table Display**: Horizontal scroll on mobile devices

### **Results Dashboard Components**

#### **1. Summary Data Table**
**Structure**:
```
+---------------------+-------------------+-------------------+-----+-------------------+
| Category            | 2G                | 3G                | ... | 5G                |
+---------------------+-------------------+-------------------+-----+-------------------+
| Base Avail.         | 98.56%            | 96.56%            | ... | 98.16%            |
|                     |                   |                   |     |                   |
| Power               | 98.78%            | 96.78%            | ... | 98.38%            |
|                     | 2500 min (45 sites)| 1800 min (32 sites)| ... | 2100 min (38 sites)|
|                     |                   |                   |     |                   |
| NOA                 | 98.65%            | 96.65%            | ... | 98.27%            |
|                     | 1800 min (32 sites)| 1500 min (28 sites)| ... | 1900 min (35 sites)|
|                     |                   |                   |     |                   |
| [Custom] Infra Issues| 98.92%           | 96.92%            | ... | 98.54%            |
|                     | 4200 min (77 sites)| 3300 min (60 sites)| ... | 4000 min (73 sites)|
+---------------------+-------------------+-------------------+-----+-------------------+
```

**Cell Format**:
- **Primary Metric**: Exclusion percentage (large font)
- **Secondary Metric**: Outage minutes (smaller font)
- **Tertiary Metric**: Affected sites in parentheses

#### **2. Trend Charts (Per Technology)**
**Configuration**:
```javascript
{
  chart: {
    type: 'line',
    zoomType: 'xy',
    height: 400
  },
  title: { text: '2G Availability Exclusion Trends (Last 10 Days)' },
  xAxis: {
    categories: dates_with_data, // Only dates with actual data
    title: { text: 'Date' }
  },
  yAxis: {
    title: { text: 'Availability %' },
    min: 97.5, // 98.5 - 1
    max: 100,
    plotLines: [{
      value: 98.5,
      color: '#2549A0',
      width: 2,
      dashStyle: 'dash',
      label: {
        text: 'Target 98.5%',
        align: 'right',
        style: { color: '#2549A0' }
      }
    }]
  },
  plotOptions: {
    line: {
      marker: { enabled: true },
      connectNulls: false // CRITICAL: Do not connect across missing dates
    }
  },
  tooltip: {
    shared: true,
    valueSuffix: '%'
  },
  legend: {
    layout: 'horizontal',
    align: 'center',
    verticalAlign: 'bottom'
  }
}
```

#### **3. Distribution Donut Charts**
**Configuration**:
```javascript
{
  chart: {
    type: 'pie',
    height: 300
  },
  title: { text: '2G Outage Distribution by Category' },
  subtitle: { text: 'Total: 15,000 minutes across 250 sites' },
  plotOptions: {
    pie: {
      innerSize: '60%',
      dataLabels: {
        enabled: false
      },
      showInLegend: true
    }
  },
  tooltip: {
    pointFormat: '{point.name}: {point.y} min ({point.percentage:.1f}%)<br/>' +
                 'Sites: {point.sites}'
  }
}
```

### **Upload Interface Components**

#### **1. Date Selection**
- **Control**: Single date picker
- **Format**: YYYY-MM-DD
- **Validation**: Cannot select future dates
- **Warning**: Alert if date already has data (overwrite confirmation)

#### **2. Availability Inputs**
```html
<div class="availability-input">
  <label for="2g-avail">Target Avail. 2G</label>
  <input type="number" 
         id="2g-avail" 
         min="0" 
         max="100" 
         step="0.01" 
         value="98.50"
         required>
</div>
```
- **Range**: 0-100
- **Precision**: Two decimal places
- **Default**: 98.5 for all technologies

#### **3. File Upload**
- **Ticket File**: Required, Excel format (.xlsx/.xls)
- **Lookup File**: Optional, uses existing if not provided
- **File Validation**: Check for required columns
- **Progress Indicator**: Real-time upload progress

#### **4. ETL Log Display**
```html
<div class="etl-log">
  <h4>Processing Log</h4>
  <div class="log-content">
    [15:30:01] Loading ticket file... ✓<br>
    [15:30:02] Adjusting temporal boundaries... ✓<br>
    [15:30:03] Calculating MTTR... ✓<br>
    [15:30:04] Filtering invalid records... ✓<br>
    [15:30:05] Processing complete: 1,250 records<br>
  </div>
  <button class="download-log">Download Log</button>
</div>
```

### **Export Functions**
1. **Results CSV**: All calculated exclusions with supporting metrics
2. **Cleaned Tickets**: Processed ticket data in CSV format
3. **Chart Images**: PNG export of individual charts
4. **Full Report**: PDF/Excel comprehensive report

---

## 🔧 API Specification (Production)

### **Authentication Endpoints**
```
POST /api/auth/login
Headers: Content-Type: application/json
Body: {
  "email": "user@iranvs.ir",  // Domain validation required
  "password": "********"
}
Response: {
  "token": "jwt_token",
  "user": { "id": 1, "email": "user@iranvs.ir", "role": "admin" }
}

POST /api/auth/logout
Headers: Authorization: Bearer {token}
Response: { "message": "Logged out successfully" }
```

### **Data Processing Endpoints**
```
POST /api/etl/process
Headers: 
  Authorization: Bearer {token}
  Content-Type: multipart/form-data
Body (form-data):
  date: "2024-01-15"
  availability_2g: 98.56
  availability_3g: 96.56
  availability_4g: 98.05
  availability_tdd: 90.00
  availability_5g: 98.16
  ticket_file: <binary file>
  lookup_file: <binary file> (optional)
Response: {
  "job_id": "uuid",
  "status": "processing",
  "estimated_time": 30,
  "log_url": "/api/etl/log/uuid"
}

GET /api/etl/status/{job_id}
Response: {
  "status": "completed",
  "processed_records": 1250,
  "cleaned_records": 1200,
  "download_url": "/api/download/cleaned/2024-01-15"
}
```

### **Data Retrieval Endpoints**
```
GET /api/availability/{date}
Response: {
  "date": "2024-01-15",
  "technologies": {
    "2G": {
      "base_availability": 98.56,
      "total_outage_seconds": 900000,
      "total_sites": 250,
      "categories": [
        {
          "name": "Power",
          "outage_seconds": 150000,
          "site_count": 45,
          "exclusion_value": 98.78
        }
      ]
    }
  }
}

GET /api/trends
Query Parameters:
  technology: "2G" (required)
  start_date: "2024-01-10" (optional, defaults to 10 days ago)
  end_date: "2024-01-20" (optional, defaults to today)
Response: {
  "dates": ["2024-01-10", "2024-01-11", ...],
  "series": [
    {
      "name": "Power",
      "data": [98.5, 98.6, null, 98.7, ...] // null for missing dates
    }
  ]
}
```

### **Export Endpoints**
```
GET /api/export/results/{date}
Headers: Accept: text/csv
Response: CSV file download

GET /api/export/tickets/{date}
Response: CSV file of cleaned tickets

GET /api/export/chart/{tech}/{chart_type}
Query Parameters: date, width, height
Response: PNG image
```

---

## 🔐 Security & Access Control

### **Authentication Requirements**
1. **Domain Restriction**: Only @iranvs.ir email addresses allowed
2. **Password Policy**: Minimum 6 characters, must contain letters and numbers
3. **Session Management**: Auto-logout after 10 minutes of inactivity
4. **Password Reset**: Forced change on first login

### **Role-Based Access Control**
| Permission | Admin | User |
|------------|-------|------|
| View Dashboard | ✓ | ✓ |
| Upload Data | ✓ | ✗ |
| Process ETL | ✓ | ✗ |
| View Results | ✓ | ✓ |
| Create Custom Categories | ✓ | ✓ |
| Export Data | ✓ | ✓ (limited) |
| User Management | ✓ | ✗ |
| System Configuration | ✓ | ✗ |

### **Security Measures**
1. **Input Validation**: All user inputs validated server-side
2. **File Upload Restrictions**: Only .xlsx and .xls files allowed
3. **SQL Injection Prevention**: Parameterized queries
4. **XSS Protection**: Output encoding for all dynamic content
5. **CSRF Protection**: Token-based for state-changing operations
6. **HTTPS Enforcement**: All communications over SSL/TLS

### **Audit Logging**
```javascript
const auditLog = {
  timestamp: "2024-01-15T10:30:00Z",
  user_id: 123,
  action: "DATA_UPLOAD",
  resource: "ticket_data",
  date_processed: "2024-01-15",
  details: {
    file_name: "tickets_20240115.xlsx",
    record_count: 1250,
    processing_time: "30 seconds"
  },
  ip_address: "192.168.1.100",
  user_agent: "Chrome/120.0.0.0"
}
```

---

## ⚙️ Business Rules & Validation

### **Data Quality Rules**
1. **Time Boundary Enforcement**: All faults adjusted to fit within target date (00:00:00 - 23:59:59)
2. **MTTR Capping**: Single ticket MTTR cannot exceed 23:59:59 (86,399 seconds)
3. **Empty Value Handling**:
   - Empty Root Cause → "BSS - Under Investigation"
   - Empty Sub Cause → "Unclear"
4. **Data Filtering**:
   - Exclude records with Aggregated_SC = "ER" (exact match, case-sensitive)
   - Exclude records with empty Site Province
5. **Category Mapping**: Unmapped causes default to "Others" category

### **Calculation Rules**
1. **Precision**: All calculations to two decimal places
2. **Zero Division**: If total outage is zero, exclusion equals base availability
3. **Value Capping**: Display values capped at 100% (even if calculation exceeds it)
4. **Missing Data**: Dates without data excluded from trend calculations (no interpolation)

### **Update & Overwrite Rules**
1. **Date-Based Overwrite**: Uploading for existing date replaces all previous data
2. **Lookup File Persistence**: Once uploaded, persists until replaced
3. **Tracker File Update**: Automatically updates for new dates, overwrites for existing

### **User Interface Rules**
1. **Terminology**: Consistent use of "Avail." instead of "TCHA"
2. **Chart Scaling**: Y-axis minimum set to 97.5% (target - 1%)
3. **Date Format**: Consistent YYYY-MM-DD format throughout
4. **Color Consistency**: Same category colors across all charts

---

## 📈 Visualization & Reporting Rules

### **Chart Generation Rules**
1. **Trend Charts**:
   - X-axis shows only dates with actual data
   - Lines do not connect across missing dates (connectNulls: false)
   - All charts include 98.5% target line
   - Legend interactive (click to hide/show series)

2. **Donut Charts**:
   - Show percentage distribution of outages
   - Include total outage minutes and site count in subtitle
   - Consistent color scheme across technologies

3. **Data Labels**:
   - Two decimal places for percentages
   - Whole numbers for minutes and site counts
   - Appropriate thousand separators for large numbers

### **Report Generation Rules**
1. **CSV Export**:
   - Include all calculated metrics
   - UTF-8 encoding
   - Column headers match UI terminology

2. **Chart Export**:
   - PNG format
   - High resolution (300 DPI for print)
   - Include title and legend

3. **Activity Reports**:
   - Daily processing summaries
   - User activity logs
   - Data quality metrics

---

## 🚀 Deployment Checklist

### **Pre-deployment Requirements**
- [ ] **Infrastructure**:
  - Web server (Nginx/Apache)
  - Python 3.8+ environment
  - PostgreSQL 12+ database
  - 10GB+ storage space
  - SSL certificate for HTTPS

- [ ] **Software Dependencies**:
  - Python packages: pandas, openpyxl, psycopg2, fastapi (or flask)
  - Node.js/npm for build tools (if applicable)
  - Highcharts license (if using commercial version)

- [ ] **Configuration Files**:
  - `SubRootcuz.xlsx` with default mappings
  - Database connection settings
  - Email server configuration
  - Logging configuration

- [ ] **Security Setup**:
  - Firewall rules configured
  - SSL/TLS certificates installed
  - Database backup strategy
  - User authentication system

### **Deployment Steps**
1. **Database Setup**:
   ```sql
   CREATE DATABASE vista_availability;
   CREATE USER vista_user WITH PASSWORD 'secure_password';
   GRANT ALL PRIVILEGES ON DATABASE vista_availability TO vista_user;
   ```

2. **Application Deployment**:
   ```bash
   # Clone repository
   git clone https://github.com/iranvs/vista-availability.git
   
   # Install dependencies
   pip install -r requirements.txt
   
   # Run database migrations
   python manage.py migrate
   
   # Start application
   gunicorn app:app --workers 4 --bind 0.0.0.0:8000
   ```

3. **Web Server Configuration** (Nginx example):
   ```nginx
   server {
       listen 80;
       server_name availability.iranvs.ir;
       return 301 https://$server_name$request_uri;
   }
   
   server {
       listen 443 ssl;
       server_name availability.iranvs.ir;
       
       ssl_certificate /path/to/certificate.crt;
       ssl_certificate_key /path/to/private.key;
       
       location / {
           proxy_pass http://localhost:8000;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
       }
   }
   ```

### **Post-deployment Verification**
1. **Functionality Tests**:
   - [ ] User authentication works
   - [ ] File upload and processing
   - [ ] Calculation accuracy
   - [ ] Chart rendering
   - [ ] Export functionality

2. **Performance Tests**:
   - [ ] ETL processing time < 60 seconds for 10,000 records
   - [ ] Page load time < 3 seconds
   - [ ] Concurrent user support (50+ users)

3. **Security Tests**:
   - [ ] SQL injection prevention
   - [ ] XSS protection
   - [ ] File upload validation
   - [ ] Session management

### **Monitoring & Maintenance**
1. **Logging**:
   - Application logs (daily rotation)
   - Access logs
   - Error logs
   - Audit logs

2. **Backup Strategy**:
   - Database: Daily backups, weekly full backups
   - Uploaded files: Daily backup
   - Configuration files: Version controlled

3. **Health Checks**:
   - Daily automated tests
   - Weekly performance review
   - Monthly security audit

---

## 🔄 Operational Procedures

### **Daily Operations**
1. **Data Upload** (performed by network operations team):
   - Collect incident tickets from previous day
   - Upload to system via web interface
   - Input actual availability metrics from NPM systems
   - Verify processing completion

2. **Review & Analysis**:
   - Review exclusion calculations
   - Identify top impacting categories
   - Generate reports for management
   - Track trends over time

3. **System Maintenance**:
   - Monitor disk space
   - Review error logs
   - Backup critical data
   - Update lookup files as needed

### **Troubleshooting Procedures**

#### **Common Issues & Solutions**
| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| **Upload fails** | File format incorrect | Verify Excel format, check required columns |
| **Calculation errors** | Invalid data in tickets | Check for null values, verify date formats |
| **Charts not loading** | Missing historical data | Upload data for missing dates |
| **Slow performance** | Large data volumes | Consider data partitioning, optimize queries |

#### **Debug Mode Activation**
```javascript
// Enable debug logging in browser console
localStorage.setItem('debug', 'true');
console.log('Debug mode enabled');

// Check available data
console.log('Available dates:', Object.keys(MOCK_DB.processedData));
```

#### **Data Validation Script**
```python
def validate_ticket_data(file_path):
    """
    Validate ticket data before processing
    """
    import pandas as pd
    
    try:
        df = pd.read_excel(file_path)
        
        # Check required columns
        required_cols = ['Fault First Occur Time', 
                        'Fault Recovery Time(Process TT_faultrecoverytime)',
                        'Site Province(Create TT_province)']
        
        missing = [col for col in required_cols if col not in df.columns]
        if missing:
            return False, f"Missing columns: {missing}"
        
        # Check data quality
        null_counts = df.isnull().sum()
        high_null = null_counts[null_counts > len(df) * 0.5]
        
        if not high_null.empty:
            return False, f"High null values in: {list(high_null.index)}"
        
        return True, "Validation passed"
        
    except Exception as e:
        return False, f"Validation error: {str(e)}"
```

---

## 🔮 Future Enhancement Roadmap

### **Phase 1: Foundation (Current)**
- ✅ Basic ETL pipeline
- ✅ Exclusion calculations
- ✅ Chart visualizations
- ✅ Data export functionality
- ✅ User authentication

### **Phase 2: Enhanced Analytics (Q2 2024)**
- [ ] **Advanced Filtering**: Filter by region, severity, time of day
- [ ] **Predictive Analytics**: ML models for outage prediction
- [ ] **Automated Reporting**: Scheduled report generation and distribution
- [ ] **Mobile Application**: Native mobile app for field teams
- [ ] **Real-time Updates**: Near-real-time data processing

### **Phase 3: Integration & Scale (Q4 2024)**
- [ ] **API Integration**: Integration with other NPM systems
- [ ] **Advanced Visualization**: 3D charts, heat maps
- [ ] **Multi-tenant Support**: Support for multiple operators
- [ ] **Data Warehouse Integration**: Integration with enterprise data warehouse
- [ ] **Advanced Security**: Two-factor authentication, role refinement

### **Phase 4: Intelligence & Automation (2025)**
- [ ] **AI-Powered Root Cause Analysis**: Automated categorization and analysis
- [ ] **Automated Recommendations**: Suggested actions based on patterns
- [ ] **Natural Language Queries**: Query system using natural language
- [ ] **Advanced Simulation**: What-if analysis with multiple variables
- [ ] **IoT Integration**: Real-time sensor data integration

---

## 📝 Usage Guidelines & Best Practices

### **For Network Operations Teams**
1. **Daily Routine**:
   - Upload ticket data by 09:00 daily
   - Verify availability metrics from source systems
   - Review exclusion calculations for anomalies
   - Export daily report for team briefings

2. **Data Quality**:
   - Ensure ticket data includes all required fields
   - Regularly update SubRootcuz file with new error codes
   - Validate time stamps before upload
   - Check for duplicate tickets

### **For Management**
1. **Performance Review**:
   - Focus on categories with largest exclusion impact
   - Track trends over weekly/monthly periods
   - Compare across technologies
   - Use custom categories for specific investigations

2. **Reporting**:
   - Use export features for management reports
   - Schedule weekly summary reports
   - Include charts in presentations
   - Track progress against 98.5% target

### **System Administration**
1. **Regular Maintenance**:
   - Daily: Check system logs, verify backups
   - Weekly: Review user accounts, clean temporary files
   - Monthly: Update software dependencies, review security logs
   - Quarterly: Full system audit, performance optimization

2. **User Management**:
   - Regular review of user accounts
   - Training for new users
   - Update permissions as roles change
   - Monitor for suspicious activity

---

## 📊 Key Performance Indicators (KPIs)

### **System Performance KPIs**
1. **Processing Speed**: < 60 seconds for 10,000 records
2. **System Availability**: 99.9% uptime
3. **User Satisfaction**: > 4.5/5 rating
4. **Data Accuracy**: 100% calculation accuracy

### **Network Performance KPIs**
1. **Availability Improvement**: Measured through exclusion calculations
2. **Category Impact**: Top 3 categories affecting availability
3. **Trend Analysis**: Direction and rate of change
4. **Target Achievement**: Days meeting 98.5% target

### **Operational KPIs**
1. **Data Completeness**: Percentage of days with data uploaded
2. **User Adoption**: Percentage of target users actively using system
3. **Report Utilization**: Number of reports generated and shared
4. **Issue Resolution**: Time to resolve system issues

---

## 🏆 Success Criteria

### **Technical Success Criteria**
- [ ] System processes 10,000+ tickets in under 60 seconds
- [ ] Calculations accurate to two decimal places
- [ ] Charts render correctly across all modern browsers
- [ ] System available 24/7 with < 0.1% downtime

### **Business Success Criteria**
- [ ] Reduced time for availability analysis by 50%
- [ ] Improved accuracy of root cause identification
- [ ] Better resource allocation based on data insights
- [ ] Consistent achievement of 98.5% availability target

### **User Success Criteria**
- [ ] 90% of target users trained and using system
- [ ] User satisfaction rating > 4.5/5
- [ ] Reduced manual reporting effort by 70%
- [ ] Positive feedback from management and operations teams

---

## 📚 Glossary

| Term | Definition |
|------|-----------|
| **Avail.** | Network Availability percentage (formerly TCHA) |
| **Exclusion** | Calculated availability when specific outage categories are excluded |
| **MTTR** | Mean Time To Repair (in seconds) |
| **Outage** | Site-minutes of downtime (MTTR × Affected Site Count) |
| **SubRootcuz** | Lookup table mapping Root/Sub causes to Categories |
| **Base Categories** | 7 predefined outage categories (Power, NOA, etc.) |
| **Custom Categories** | User-defined combinations of base categories |
| **Target Line** | Reference line at 98.5% in all charts |
| **ETL** | Extract, Transform, Load data processing pipeline |

---

## 📞 Support & Contact

### **Technical Support**
- **Email**: support.availability@iranvs.ir
- **Hours**: 09:00-16:00 (Iran Standard Time), Saturday-Wednesday
- **Emergency**: +98-21-XXXXXXX (Network Operations Center)

### **Training & Documentation**
- **User Manual**: Available in system help section
- **Training Sessions**: Weekly on Tuesdays at 10:00
- **Video Tutorials**: Available on internal portal
- **Quick Start Guide**: One-page reference sheet

### **Feedback & Improvement**
- **Feedback Form**: Available in system settings
- **Improvement Requests**: Submit via email to product team
- **Bug Reports**: Use system reporting tool or email support

---

*Document Version: 2.0 | Last Updated: [02/02/2026] | Classification: Internal Use Only | For: Vista Services Platform Team*

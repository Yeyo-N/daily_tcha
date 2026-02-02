# Vista Service Daily TCHA Exclusion System

## 📋 Project Overview

**Vista Service Daily TCHA Exclusion** is a web-based dashboard application designed for telecommunications network performance monitoring and management. The system enables users to analyze and exclude TCHA outages across different network technologies (2G, 3G, 4G, TDD, 5G) while categorizing incidents by root causes.

### **Core Purpose**
Transform raw incident ticket data into actionable insights by calculating TCHA exclusions based on outage categories, enabling accurate network availability reporting.

---

## 🎯 Key Features

### **1. Authentication & Security System**
- **Role-based access control** (Admin/User roles)
- **Secure login** with @iranvs.ir domain restriction
- **Password policies** with forced password change on first login
- **Auto-logout** after 10 minutes of inactivity
- **Account status management** (active/inactive)

### **2. Admin Dashboard**
- **Real-time system statistics** display
- **Activity logging** with date filtering capabilities
- **User management** with CRUD operations
- **Export functionality** for logs and user activities

### **3. Data Upload & ETL Processing**
- **Excel file upload** for incident ticket data
- **Technology-specific TCHA input** (2G, 3G, 4G, TDD, 5G) availability input in number only format with two decimals
- **Simulated ETL pipeline** with real-time logging and auto download log file when completed, logs are very comprehensive
- **SubRootcuz.xlsx lookup file** integration, no need to upload at each attempt. irf not uploaded use the previous one then
- **Automated outage calculation** based on input of TCHA

### **4. Results Visualization**
- **Technology-wise outage summary tables**
- **Interactive Highcharts donut charts** for outage distribution
- **Custom row creation** for combined category analysis
- **Date-based data filtering**
- **Responsive grid layout** for multi-technology comparison
- **Interactive line charts** for showing availability per technology for the time window of slected date and its past 10 days.

### **5. Data Analysis Features**
- **Categorized outage analysis** (Power, NOA, Non Telecom, PA/UA, Spare, Third Party, Others)
- **Technology-specific algorithms** for outage distribution
- **Custom exclusion rows** for combined category analysis
- **Export capabilities** for charts and table results and cleaned ticket data which calculation is based on

---

## 🏗️ Architecture & Technologies

### **Frontend Stack**
- **Pure HTML/CSS/JavaScript** (No external frameworks)
- **Highcharts.js** for data visualization
- **CSS Grid & Flexbox** for responsive layouts
- **Modular JavaScript** architecture with clear separation of concerns

### **Application Architecture**
```
App
├── Auth Controller (Authentication & Authorization)
├── Views Controller (UI Rendering)
├── ETL Service (Data Processing)
├── Charts Service (Visualization)
├── Actions Controller (Business Logic)
├── Modals Controller (Dialog Management)
└── Logger Service (Activity Tracking)
```

### **Mock Database Structure**
```javascript
MOCK_DB = {
  users: [],      // User accounts with roles
  logs: [],       // System activity logs
  processedData: {} // Date-keyed analysis results
}
```

---

## 🎨 UI/UX Design Features

### **Professional Navigation**
- **Custom tabbed interface** with visual separators
- **Role-specific navigation** (Admin sees all tabs, User sees only Results)
- **Active state highlighting** with bottom border indicators
- **User profile display** with role badge

### **Dashboard Components**
- **Stat cards** with hover effects
- **Collapsible ETL log display**
- **Modal dialogs** with backdrop blur
- **Toast notifications** for system feedback
- **Responsive tables** with horizontal scrolling

### **Color Scheme**
- **Primary Blue**: `#2549A0` (Vista brand color)
- **Accent Green**: `#78DD12` (Success/actions)
- **Neutral Grays**: For backgrounds and text
- **Technology-specific colors** for chart differentiation

---

## 🔧 Technical Implementation

### **Key Algorithms**
1. **Outage Distribution Algorithm**
   - Technology-specific weighting factors
   - Category-based outage calculation
   - TCHA-target influenced results

2. **ETL Simulation**
   - Step-by-step process visualization
   - Mock data generation with realistic distributions
   - Technology-characteristic modeling

3. **Chart Generation**
   - Consistent color mapping across technologies
   - Percentage-based donut charts
   - Interactive tooltips with detailed data

### **State Management**
- **Global STATE object** for session management
- **Date-based data segregation**
- **User role persistence**

### **Security Features**
- **Input validation** on all forms
- **Activity logging** for audit trails
- **Session timeout** with user activity tracking
- **Role-based view restrictions**

---

## 📊 Data Flow

```
1. Admin Uploads Data
   ↓
2. Admin Uploads TCHA per Technology
   ↓
3. ETL Process Simulates Data Processing
   ↓
4. System Generates Outage Distribution Models
   ↓
5. Results Displayed in Tables & Charts
   ↓
6. Users Can Add Custom Exclusion Rows
   ↓
7. Final Results Available for Export
```

---

## 🚀 Deployment & Development

### **Development Notes**
- **Single HTML file** deployment
- **No external dependencies** beyond Highcharts CDN
- **In-memory database** for prototype purposes
- **Ready for backend integration** with REST API

### **Future Integration Points**
```yaml
Backend Integration:
  - PostgreSQL database
  - Docker containerization
  - REST API endpoints
  - Real file upload processing
  
Production Features:
  - Actual Excel file parsing
  - Database persistence
  - User authentication server
  - Scheduled ETL jobs
```

### **Demo Credentials**
```
Admin: admin@iranvs.ir / 1qaz!QAZ
User:  user@iranvs.ir / 1qaz!QAZ
```

---

## 📈 Business Value

### **For Network Operations**
- **Accurate TCHA reporting** with proper exclusions
- **Root cause analysis** of network outages
- **Technology performance comparison**
- **Historical trend analysis**

### **For Management**
- **Single source of truth** for network availability
- **Standardized reporting** across technologies
- **Audit trail** of all calculations and changes
- **Customizable analysis** with combined categories

---

## 🔮 Future Enhancements

### **Planned Features**
1. **Real database integration** with PostgreSQL
2. **Actual Excel file processing** (xlsx parsing)
3. **Automated scheduled reports**
4. **Multi-tenant support** for different regions
5. **Advanced analytics** with trend prediction
6. **Mobile-responsive design** improvements
7. **API endpoints** for third-party integration

### **Scalability Considerations**
- **Modular architecture** ready for component extraction
- **Service separation** potential (Auth, ETL, API)
- **Cloud deployment** ready with Docker configuration
- **Performance optimization** for large datasets

---

## 📝 Conclusion

The **Vista Service Daily TCHA Exclusion System** represents a comprehensive solution for telecommunications network performance analysis. By combining intuitive data upload, sophisticated ETL simulation, and powerful visualization tools, it empowers NPM teams to accurately calculate and report network availability while identifying key areas for operational improvement.

The prototype demonstrates full functionality with a clean, professional interface and is production-ready with backend integration.

---

*Last Updated: Prototype v1.0 | For NPM Team Use | Vista Services Platform*

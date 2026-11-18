# PaceCraft: The Split Optimizer — System Diagrams

## 1. Context Diagram
```mermaid
flowchart LR
  R["Amateur Runner"]
  C["Running Coach"]
  S(["0.0 PaceCraft Split Optimizer System"])
  G["Garmin Sponsor Platform"]

  R -->|race goal details| S
  R -->|access identifier| S
  S -->|mile split schedule| R
  S -->|safety rule message| R
  S -->|access identifier receipt| R
  C -->|review decision| S
  S -->|plan review list| C
  S -->|watch ready split file| G
  G -->|pacing metric standard| S
```

## 2. Level-0 Data Flow Diagram
```mermaid
flowchart TB
  R["Amateur Runner"]
  C["Running Coach"]
  G["Garmin Sponsor Platform"]

  P1(["1.0 Issue And Restore Access Identifier"])
  P2(["2.0 Calculate Split Schedule"])
  P3(["3.0 Validate And Save Pacing Plan"])
  P4(["4.0 Review And Update Plan Status"])
  P5(["5.0 Search And Filter Plans"])
  P6(["6.0 Produce Summary Report"])

  D1["D1 pacecraft_plans"]
  D2["D2 pacecraft_users"]
  D3["D3 pacecraft_reviews"]

  R -->|access identifier| P1
  P1 -->|user record| D2
  D2 -->|stored user record| P1
  P1 -->|access identifier receipt| R

  R -->|race goal details| P2
  G -->|pacing metric standard| P2
  P2 -->|draft split schedule| P3
  P2 -->|mile split schedule| R

  P3 -->|validated plan record| D1
  P3 -->|safety rule message| R
  P3 -->|watch ready split file| G

  D1 -->|stored plan record| P4
  C -->|review decision| P4
  P4 -->|updated plan status| D1
  P4 -->|review log entry| D3
  P4 -->|plan review list| C

  R -->|search text| P5
  C -->|search text| P5
  D1 -->|stored plan record| P5
  P5 -->|matching plan list| R
  P5 -->|matching plan list| C

  D1 -->|stored plan record| P6
  D3 -->|review log entry| P6
  P6 -->|summary totals| R
  P6 -->|summary totals| C
```

## 3. Level-1 Data Flow Diagram: Process 3.0 (Validate and Save Pacing Plan)
```mermaid
flowchart TB
  R["Amateur Runner"]
  G["Garmin Sponsor Platform"]
  IN["From 2.0 Calculate Split Schedule"]

  P31(["3.1 Check Required Entry Fields"])
  P32(["3.2 Apply Safety Pace Floor BR-01"])
  P33(["3.3 Apply Marathon Fatigue Rule BR-02"])
  P34(["3.4 Write Plan Record And Set Status"])
  P35(["3.5 Format Watch Ready Split File"])

  D1["D1 pacecraft_plans"]
  D2["D2 pacecraft_users"]

  IN -->|draft split schedule| P31
  R -->|race goal details| P31
  P31 -->|entry error message| R
  P31 -->|checked plan details| P32

  P32 -->|safety rule message| R
  P32 -->|approved pace plan| P33

  P33 -->|fatigue rule message| R
  P33 -->|classified plan details| P34

  D2 -->|stored user record| P34
  P34 -->|validated plan record| D1
  P34 -->|saved plan confirmation| R
  P34 -->|classified plan details| P35
  P35 -->|watch ready split file| G
```

## 4. Level-1 Data Flow Diagram: Process 4.0 (Review and Update Plan Status)
```mermaid
flowchart TB
  C["Running Coach"]
  R["Amateur Runner"]

  P41(["4.1 Retrieve Plans Awaiting Review"])
  P42(["4.2 Check Reviewer Role BR-03"])
  P43(["4.3 Record Approval Or Flag Decision"])
  P44(["4.4 Write Review Log Entry"])

  D1["D1 pacecraft_plans"]
  D3["D3 pacecraft_reviews"]

  D1 -->|stored plan record| P41
  P41 -->|plan review list| C
  P41 -->|selected plan record| P42

  C -->|review decision| P42
  P42 -->|authority rule message| R
  P42 -->|authorised review decision| P43

  P43 -->|updated plan status| D1
  P43 -->|status change notice| C
  P43 -->|decision detail| P44
  P44 -->|review log entry| D3
  D3 -->|review log entry| P41
```

## 5. Business Process Model: Create and Approve a Race Strategy
```mermaid
flowchart TB
  START(["Start. Runner decides to race"]) --> T1
  T1["Manual. Choose a goal finish time and race distance"] --> T2
  T2["System-supported. Issue or restore access identifier, DFD 1.0"] --> T3
  T3["Manual. Enter name, distance, goal time and strategy"] --> T4
  T4["System-supported. Calculate average pace and mile splits, DFD 2.0"] --> T5
  T5["System-supported. Apply safety pace floor and fatigue rule, DFD 3.1 to 3.3"] --> D1

  D1{"Exclusive decision. Is the average pace safe under BR-01?"}
  D1 -->|"No. Pace faster than four thirty per mile"| EX1
  D1 -->|Yes| T6

  EX1["Exception path. System-supported. Block the save and display the BR-01 safety message, DFD 3.2"] --> EX2
  EX2["Manual. Runner raises the goal finish time"] --> T3

  T6["System-supported. Save the plan as Draft or Flagged, DFD 3.4"] --> T7
  T7["Manual. Coach opens the strategy hub before race day"] --> T8
  T8["System-supported. Check reviewer role under BR-03, DFD 4.2"] --> D2

  D2{"Is the reviewer acting as a running coach?"}
  D2 -->|No| EX3
  D2 -->|Yes| T9
  EX3["System-supported. Refuse the approval and show the BR-03 message, plan stays Draft, DFD 4.2"] --> ENDB

  T9["System-supported. Set status to Coach Approved or Flagged and write the review log, DFD 4.3 and 4.4"] --> T10
  T10["System-supported. Show the summary report and export the watch ready splits, DFD 6.0 and 3.5"] --> T11
  T11["Manual. Runner saves the access identifier and runs the race to plan"] --> ENDA

  ENDA(["End. Approved plan on race day"])
  ENDB(["End. Plan returned to the runner as an unapproved draft"])
```

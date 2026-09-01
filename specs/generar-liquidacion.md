# Feature Specification: Generate Settlement

**Created**: 2026-08-28  

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Generate a booking's settlement at check-out (Priority: P1)

As a front-desk agent, once a guest's stay ends (check-out), I want to generate the booking's settlement so I get the consolidated financial breakdown (lodging, OTA commission if applicable, and taxes) and know the actual net income the hotel receives.

**Why this priority**: This is the core flow of Module 3: without this capability there is no financial settlement, and the hotel cannot close any stay from an accounting standpoint. It is the module's minimum viable slice.

**Independent Test**: Can be fully tested by completing check-out on a direct-channel booking and verifying the system delivers the breakdown (Base Lodging, Taxes, Net Income) without needing the other stories.

**Acceptance Scenarios**:

1. **Scenario**: Settlement for a direct-channel booking
   - **Given** a booking with completed check-out and "Direct Channel" origin
   - **When** the front-desk agent generates the settlement
   - **Then** the system calculates the Base Lodging (season rate x nights), applies the corresponding tax, and presents the Net Income with no commission deducted

2. **Scenario**: Settlement for a booking originating from an OTA
   - **Given** a booking with completed check-out, "OTA" origin with an external confirmation code and an agreed commission percentage on record
   - **When** the front-desk agent generates the settlement
   - **Then** the system calculates the Base Lodging, deducts the OTA Commission (Lodging Value x Commission %), applies the tax, and presents the Net Income reflecting the commission deduction

---

### User Story 2 - View a settlement, at check-in and after check-out (Priority: P2)

As a front-desk agent, I want to request the settlement at check-in to see the expected breakdown before the stay begins, and as a finance administrator I want to view and export the detail of an already generated settlement after check-out, so both the guest-facing price and the accounting record stay consistent.

**Why this priority**: Viewing/requesting the settlement loses business value if it cannot be checked before the stay or audited afterward; however, the post-check-out view depends on a settlement already existing (Story 1), which is why it ranks second.

**Independent Test**: The check-in scenario can be tested on its own right after check-in, with no settlement generated yet, by requesting the settlement and verifying the system returns the expected breakdown from booking data. The post-check-out scenario can be tested by first generating a settlement (Story 1) and then viewing its detail and exporting it, verifying the amounts match exactly what was calculated at generation time.

**Acceptance Scenarios**:

1. **Scenario**: Requesting the settlement at check-in
   - **Given** a booking with completed check-in, its rate/season data, and OTA commission percentage (if applicable) on record
   - **When** the front-desk agent requests the settlement for that booking
   - **Then** the system shows the expected breakdown (Base Lodging, OTA Commission if applicable, Taxes) calculated from the data known at check-in

2. **Scenario**: Viewing the detail of an existing settlement
   - **Given** a settlement previously generated for a booking
   - **When** the finance administrator looks up the booking and requests to view its settlement
   - **Then** the system shows the complete breakdown (Base Lodging, OTA Commission if applicable, Taxes, Net Income) exactly as calculated

3. **Scenario**: Exporting a settlement
   - **Given** a previously generated settlement
   - **When** the finance administrator requests to export it
   - **Then** the system delivers a document with the settlement's complete financial breakdown

---

### Edge Cases

- What happens if someone attempts to generate the settlement for a booking whose check-out has not been completed?
- How does the system handle an OTA-origin booking that does not have an agreed commission percentage on record?
- What happens if the stay spans both high and low season dates, requiring different nightly rates within the same settlement?
- How is a booking settled in case of an early check-out (departure before the scheduled date) or a no-show?
- What happens if someone attempts to generate the settlement more than once for the same booking?
- How does the system handle a booking with multiple rooms associated with a single guest account?
- What happens if the guest or booking is tax-exempt under local legislation?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST calculate Base Lodging as the applicable season rate multiplied by the number of nights of the stay.
- **FR-002**: The system MUST identify the booking's origin channel (Direct or OTA) to determine whether an intermediary commission applies.
- **FR-003**: The system MUST calculate and deduct the OTA Commission (Lodging Value x agreed commission %) when the booking originates from an intermediary.
- **FR-004**: The system MUST apply the tax according to the rate currently in effect under local legislation on the total services rendered.
- **FR-005**: The system MUST present the complete settlement breakdown (Base Lodging, OTA Commission, Taxes, Net Income) once the calculation is complete.
- **FR-006**: The system MUST prevent generating the settlement for a booking whose check-out has not been completed.
- **FR-007**: The system MUST persist each generated settlement, linked to its booking, for later review by accounting/finance administration.
- **FR-008**: The system MUST allow exporting a generated settlement's breakdown in a format reviewable outside the system.
- **FR-009**: The system MUST allow requesting the settlement at check-in, computing the expected breakdown (Base Lodging, OTA Commission if applicable, Taxes) from the booking's rate, season, and commission data known at that point, without requiring check-out to have occurred.
- **FR-010**: When an OTA-origin booking has no agreed commission percentage on record, the system MUST flag the settlement as pending missing data instead of generating it with an assumed or default commission value.
- **FR-011**: The system MUST apply the tax rate as set by the government under local legislation (see "Consultar porcentaje de IVA" in the Módulo 3 use-case diagram, actor Gobierno); the hotel does not define, configure, or override this rate — the system only consumes the externally published percentage in effect.

### Key Entities *(include if feature involves data)*

- **Settlement**: Financial document generated from a booking; represents the consolidated breakdown of Base Lodging, OTA Commission (if applicable), Taxes, and Net Income. Linked to a single booking.
- **Booking**: Represents the guest's stay; supplies the settlement with the stay dates, origin channel (Direct/OTA), external confirmation code (if applicable), and agreed commission percentage.
- **Season Rate**: Base nightly price in effect according to the season calendar (high/low) applied to the Base Lodging calculation.
- **OTA Commission**: Percentage agreed with the intermediary (OTA) that is deducted from the lodging value when the booking originates from that channel.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: The front-desk agent can generate the settlement of a completed booking in under 1 minute from the moment it is requested.
- **SC-002**: 100% of settlements generated for OTA bookings reflect the deducted commission calculated correctly according to the agreed percentage on record.
- **SC-003**: The finance administrator can reconcile a booking's net income with the corresponding OTA's reports without manually recalculating any value.
- **SC-004**: Fewer than 1% of generated settlements require correction due to calculation errors.

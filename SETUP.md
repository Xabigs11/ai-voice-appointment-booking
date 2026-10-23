# Setup

## Requirements

- An n8n instance with HTTPS webhooks.
- Google Calendar OAuth credentials in n8n.
- Google Sheets OAuth credentials in n8n.
- Gmail OAuth credentials in n8n.
- A Vapi assistant with two Function tools.

## 1. Import the n8n workflows

Import both JSON files from `workflows/`. The public exports are inactive and do not contain credential references or private resource identifiers.

## 2. Connect external resources

In `Check Calendar Availability`:

1. Connect Google Calendar credentials to `Comprobar disponibilidad Calendar`.
2. Select the Calendar that contains the business availability.

In `Book Appointment`:

1. Connect Google Calendar credentials to `Recomprobar horario seleccionado` and `Crear reunión en Calendar`.
2. Select the same Calendar in both nodes.
3. Connect Google Sheets credentials to `Guardar reserva en CRM` and `Registrar email enviado en CRM`.
4. Select the destination spreadsheet and sheet tab in both nodes.
5. Connect Gmail credentials to `Enviar confirmación al cliente`.

The CRM sheet must expose the columns referenced by the Google Sheets nodes, including `booking_key`, `client_name`, `client_email`, `appointment_date`, `start_time`, `calendar_event_id`, `meet_link`, `confirmation_email_status`, and their related status fields.

## 3. Activate the production webhooks

Activate both workflows and copy their production webhook URLs. Do not publish those URLs in a public repository.

Before using the workflows outside a controlled demo, configure webhook authentication with a shared secret or authenticated header and restrict requests to the expected Vapi integration. The public templates intentionally omit credentials.

The sanitized exports use these local path labels:

- `portfolio-check-calendar-availability`
- `portfolio-book-appointment`

Your imported n8n instance will generate the complete production URLs.

## 4. Configure the Vapi tools

Create two Function tools in Vapi using the definitions in `vapi/` and set their server URLs to the corresponding production webhooks. The public tool files contain named URL placeholders that must be replaced before use.

### `consultar_disponibilidad`

Required arguments:

- `date`: requested date in `YYYY-MM-DD` format.
- `timePreference`: one of `morning`, `afternoon`, or `any`.

### `reservar_cita`

Required arguments:

- `clientName`
- `clientEmail`
- `businessName`
- `service`
- `projectSummary`
- `date`: selected date in `YYYY-MM-DD` format.
- `startTime`: selected local time in `HH:MM` 24-hour format.

The booking tool must only be called after the assistant summarizes the request and receives explicit confirmation from the user.

## 5. Configure the Vapi assistant

Use `vapi/assistant.public.json` as the reproducible assistant template. Before applying it in Vapi:

1. Select an ElevenLabs voice and replace `YOUR_ELEVENLABS_VOICE_ID`.
2. Create both Function tools and replace `YOUR_CHECK_AVAILABILITY_TOOL_ID` and `YOUR_BOOK_APPOINTMENT_TOOL_ID` with their generated IDs.
3. Confirm in the dashboard that `consultar_disponibilidad` and `reservar_cita` are mapped to the correct tool references.
4. Keep the Soniox transcriber configured for Spanish (`es`).

The public file contains placeholders and is not ready to send to the Vapi API until those values have been replaced.

## 6. Test the three critical paths

1. A valid request with a free slot.
2. An invalid request that must stop before Calendar.
3. A previously offered slot that becomes occupied before confirmation.

Use fictional data during testing and verify that no duplicate CRM record is created.

For strict concurrency guarantees, add a lock or transactional reservation layer around the final availability check and Calendar event creation. The included recheck reduces stale-slot bookings but cannot make two separate Calendar API operations atomic.


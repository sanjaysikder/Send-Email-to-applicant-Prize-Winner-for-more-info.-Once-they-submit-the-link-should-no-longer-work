# Applicant/ Prize Winner Notification & Registration (Send Email to applicant/Prize Winner for more info. Once they submit, or if the status is not “Requested”, the link should no longer work.
## Overview
This Oracle APEX application automates the process of notifying prize winners and collecting their information through a secure self-service portal. The system ensures that only winners with a "Requested" status can access the registration form, and the link becomes invalid once the information is submitted.

## Table of Contents
- Step 1: Send Email Process
- Step 2: Create Public Registration Page
- Step 3: Create Invalid URL Page
- Step 4: Implement Status Check

## Step 1: Send Email Process

``` Send Email Process:

DECLARE
    l_body_html  CLOB;
    v_email_id   NUMBER;
    v_PRIZE_WINNER_LINE_ID number;
    v_name       NVARCHAR2(200);
    v_channel_name NVARCHAR2(200);
    v_program    NVARCHAR2(200);
    v_email_id_txt NVARCHAR2(200);
    v_INTERFACE_STATUS NVARCHAR2(200);
BEGIN

     UPDATE XXDMI_PRIZE_WINNER_LINES 
     SET INTERFACE_STATUS = 'Requested',
         REQUESTED_DATE = SYSDATE,
         REQUESTED_BY = :APP_USER_ID
     WHERE PRIZE_WINNER_LINE_ID = :P9_PRIZE_WINNER_LINE_ID;

    -- Fetch winner details
    SELECT A.PRIZE_WINNER_LINE_ID, A.WINNER_FIRST_NAME || ' ' || A.WINNER_LAST_NAME,
           B.ORGANIZATION_EN,
           C.PROGRAM_NAME,
           A.CONTACT_EMAIL,
           A.INTERFACE_STATUS
      INTO v_PRIZE_WINNER_LINE_ID, v_name, v_channel_name, v_program, v_email_id_txt, v_INTERFACE_STATUS
      FROM XXDMI_PRIZE_WINNER_LINES A
           JOIN DEPARTMENT_MASTER B
             ON A.CHANNEL_ID = B.ORGANIZATION_ID
           JOIN PROGRAM_MASTER C
             ON A.PROGRAM_ID = C.PROGRAM_ID
     WHERE A.PRIZE_WINNER_LINE_ID = :P9_PRIZE_WINNER_LINE_ID;

    -- Construct email HTML
    l_body_html := '
<div style="width:820px;  margin:auto; border:1px solid #999;padding: 20px;  font-family:Tahoma, Arial, sans-serif;">

  <!-- HEADER with logo -->
  <div style="width:800px;  margin:auto; padding:15px; background-color:#ffffff;">
    <table width="100%" cellpadding="0" cellspacing="0">
      <tr>
        <!-- Logo -->
        <td style="width:150px; text-align:left;">
          <img src="https://via.placeholder.com/120x50?text=LOGO" alt="LOGO" style="display:block;">
        </td>
        <!-- Title -->
        <td style="text-align:center; font-size:22px; font-weight:bold; color:#a02a2a;">
          Congratulations / تهنئة
        </td>
      </tr>
    </table>
  </div>
  <!-- INNER CONTENT WRAPPER with border and padding -->
  <div style="width:800px; margin:auto;">
    <hr style="border: none; border-top: 1px solid #8b3f00;">
    <!-- Arabic Section -->
    <div style="border:1px solid #999; padding: 10px; text-align:right; direction:rtl; border-bottom:1px solid #999;">
      <p style="margin:6px 0;">عزيزي/ة <strong>' || v_name || '</strong>،</p>
      <p style="margin:6px 0;">أنت الفائز في برنامج تلفزيوني <strong>"' || v_program || '"</strong>.</p>
      <p style="margin:6px 0;">يرجى تزويدنا ببياناتك عبر <a href="https://oracleapex.com/ords/r/live_demo/prize-winners-automation/winner-registration-self-service?P10_PRIZE_WINNER_LINE_ID=' || v_PRIZE_WINNER_LINE_ID || '&P10_INTERFACE_STATUS=' || v_INTERFACE_STATUS || '"
style="color:#7a3df0; text-decoration:underline;">
Winner Details
</a> لإرسال الجائزة.</p>
      <p style="margin:6px 0;">يمكنك استلام الجائزة خلال ستة أشهر من تاريخ الإخطار.</p>
      <p style="margin:6px 0;">للاستفسار: Ravindar.perela@gmail.com</p>
      <p style="margin:6px 0;">مع التحية،<br>Ravindar Perela<br>www.ravindar.com</p>
    </div>
    <div style="height:20px;"></div>
    <!-- English Section -->
    <div style="border:1px solid #999; padding: 10px; text-align:left; direction:ltr;">
      <p style="margin:6px 0;">Dear ' || v_name || ',</p>
      <p style="margin:6px 0;">Congratulations. You are the winner of ' || v_channel_name || ' show <strong>"' || v_program || '"</strong>.</p>
      <p style="margin:6px 0;">Please provide your details at <a href="https://oracleapex.com/ords/r/live_demo/prize-winners-automation/winner-registration-self-service?P10_PRIZE_WINNER_LINE_ID=' || v_PRIZE_WINNER_LINE_ID || '&P10_INTERFACE_STATUS=' || v_INTERFACE_STATUS || '"
style="color:#7a3df0; text-decoration:underline;">Winner Details</a> to claim your prize.</p>
      <p style="margin:6px 0;">Prizes can be claimed within six months of the date of notification.</p>
      <p style="margin:6px 0;">If you have any queries, please email: Ravindar.perela@gmail.com</p>
      <p style="margin:6px 0;">Sincerely,<br>Ravindar Perela<br>Website: www.ravindar.com</p>
    </div>

  </div>

</div>
';

    -- Send email
    v_email_id := APEX_MAIL.SEND(
        p_to        => v_email_id_txt,
        p_from      => 'sanjaysikder71@gmail.com',
        p_body      => l_body_html,
        p_body_html => l_body_html,
        p_subj      => 'Congratulations. You are the winner of ' || v_channel_name || ' show "' || v_program || '"'
    );

    COMMIT;


END;

```



## Step 2: Create Public information Update/Registration Page

- Create a Public page with related Field that Applicant input theis information. and Create a Information update process.

- Add page Inline Css to hide Application Navigation menu or others:
  
```css
.t-Header-nav {
    display: none;  
}

.t-Body-nav {
    display: none;
}

.t-Header {
    display: none;
}
```


 - Create Update Process:
 
 ```update process
 BEGIN
UPDATE XXDMI_PRIZE_WINNER_LINES SET 
WINNER_FIRST_NAME = :P10_WINNER_FIRST_NAME,
WINNER_MIDDLE_NAME = :P10_WINNER_MIDDLE_NAME,
WINNER_LAST_NAME = :P10_WINNER_LAST_NAME,
RELATION = :P10_RELATION,
CONTACT_NUMBER = :P10_CONTACT_NUMBER,
CONTACT_EMAIL = :P10_CONTACT_EMAIL,
COUNTRY_NAME = :P10_COUNTRY_NAME,
INTERFACE_STATUS = 'Received'

WHERE PRIZE_WINNER_LINE_ID= :P10_PRIZE_WINNER_LINE_ID;

--apex_application.g_print_success_message :=  'Winner information successfully submitted.';

END;
```


## Step 3: Create another public page for for check status and redirect to Error message (Invalid URL) page.


- AAdd this PL/SQL code to a "Blank with Attributes" region:
- 
```error message

begin
    htp.p('<div align="center" style="font-weight: bold; color:red; font-size:17px; padding:80px; text-align:center; border:1px solid #ccc; height:70px; margin:50px auto; width:600px;">
        <h2>Invalid URL</h2>
        <p>The registration link you have used is invalid or has expired.</p>
        <p>Please contact support if you believe this is an error.</p>
    </div>');
    
    htp.p('<div style="text-align:center; margin-top:20px;">
        <br><br><br><br><br><br>
        <p>For assistance, please email: Ravindar.perela@gmail.com</p>
    </div>');
end;


```

## Step 4: Create a "Before Header" process on the information Update/Registration Page that validates the winner's status before displaying the form.

- Before Header Process

```plsql Process 

DECLARE
    l_status XXDMI_PRIZE_WINNER_LINES.INTERFACE_STATUS%TYPE;
BEGIN
    SELECT INTERFACE_STATUS
    INTO   l_status
    FROM   XXDMI_PRIZE_WINNER_LINES
    WHERE  PRIZE_WINNER_LINE_ID = :P10_PRIZE_WINNER_LINE_ID;

    IF l_status <>  'Requested' THEN
        apex_util.redirect_url(
            p_url => 'https://oracleapex.com/ords/r/live_demo/prize-winners-automation/invalid-url?session='
                     || v('APP_SESSION')
        );
    END IF;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        NULL; -- do nothing
END;


```

## System Workflow
- 1. Admin triggers email process for a prize winner
- 2. System updates status to "Requested" and sends email with unique link
- 3. Winner clicks link and is redirected to registration page
- 4. System validates status before displaying form
- 5. Winner submits information → Status updates to "Received"
- 6. Link becomes invalid for any future access
- 7. Invalid URL page displays for expired/completed registrations


 # Thank you
 ## Sanjay Sikder

 You can connect with me on [LinkedIn](https://www.linkedin.com/in/sanjay-sikder/)!

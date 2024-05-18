# Assignment
TeachMint Automation Script
This script automates the process of logging into TeachMint using phone number and OTP, and generating a School Leaving Certificate for a student.

Prerequisites
Python 3.x
Selenium WebDriver
Chrome WebDriver
webdriver_manager (optional but recommended for managing WebDriver)
Installation
Install Python 3.x from python.org
Install required dependencies:
bash
Copy code
pip install selenium
pip install webdriver-manager
Usage
Replace username and password in the script with your TeachMint credentials.
Run the script:
bash
Copy code
python script_name.py
Script Overview
The script performs the following tasks:

Logs into TeachMint using phone number and OTP.
Navigates to the Certificates section and generates a School Leaving Certificate for a specified student.
Example
python
Copy code
from selenium import webdriver
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.chrome.service import Service as ChromeService
from selenium.webdriver import Chrome
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import Select
import time

# Replace 'username' and 'password' with actual credentials
username = "your_username"
password = "your_password"

# Default and headless options for Chrome WebDriver
default_options = [
    "--disable-extensions",
    "--disable-user-media-security=true",
    "--allow-file-access-from-files",
    "--use-fake-device-for-media-stream",
    "--use-fake-ui-for-media-stream",
    "--disable-popup-blocking",
    "--disable-infobars",
    "--enable-usermedia-screen-capturing",
    "--disable-dev-shm-usage",
    "--no-sandbox",
    "--auto-select-desktop-capture-source=Screen 1",
    "--disable-blink-features=AutomationControlled"
]

headless_options = ["--headless", "--use-system-clipboard", "--window-size=1920x1080"]

def browser_options(chrome_type):
    webdriver_options = webdriver.ChromeOptions()
    notification_opt = {"profile.default_content_setting_values.notifications": 1}
    webdriver_options.add_experimental_option("prefs", notification_opt)
    if chrome_type == "headless":
        var = default_options + headless_options
    else:
        var = default_options
    for d_o in var:
        webdriver_options.add_argument(d_o)
    return webdriver_options

def get_webdriver_instance(browser=None):
    base_url = "https://accounts.teachmint.com/"
    caps = DesiredCapabilities().CHROME
    caps["pageLoadStrategy"] = "normal"
    driver = Chrome(service=ChromeService(ChromeDriverManager().install()),
                    options=browser_options(browser))
    driver.command_executor._commands["send_command"] = ("POST", '/session/$sessionId/chromium/send_command')
    driver.maximize_window()
    driver.get(base_url)
    set_driver(driver)
    return driver

def enter_phone_number_otp(driver, creds):
    driver.find_element_by_xpath("//input[@type='text']").send_keys(creds[0])
    time.sleep(1)
    print("Entered user phone number {}".format(creds[0]))
    driver.find_element_by_id("send-otp-btn-id").click()
    WebDriverWait(driver, 30).until(EC.invisibility_of_element((By.CSS_SELECTOR, "loader")))
    WebDriverWait(driver, 30).until(EC.invisibility_of_element((By.CLASS_NAME, "loader")))
    time.sleep(1)
    _input_otp_field = "//input[@data-group-idx='{}']"
    for i, otp in enumerate(creds[1]):
        otp_field = _input_otp_field.format(str(i))
        write(otp, into=S(otp_field))
        print("Entered OTP {}".format(creds[1]))
    time.sleep(1)
    driver.find_element_by_id("submit-otp-btn-id").click()
    time.sleep(2)
    driver.find_element_by_xpath("//span[@onclick='onSkipPassCreationClick()']").click()
    WebDriverWait(driver, 30).until(EC.invisibility_of_element((By.CSS_SELECTOR, "loader")))
    WebDriverWait(driver, 30).until(EC.invisibility_of_element((By.CLASS_NAME, "loader")))
    time.sleep(1)
    print("Successfully entered user phone number and OTP")


def login(admin_credentials=["0000020232", "120992", "@Automation-2"],
          account_name="@Automation-2"):
    driver = get_webdriver_instance()
    WebDriverWait(driver, 30).until(EC.invisibility_of_element((By.CSS_SELECTOR, "loader")))
    WebDriverWait(driver, 30).until(EC.invisibility_of_element((By.CLASS_NAME, "loader")))
    time.sleep(1)
    enter_phone_number_otp(driver, admin_credentials)
    user_name = f"//div[@class='profile-user-name']/..//div[text()='{account_name}']"
    WebDriverWait(driver, 30).until(EC.element_to_be_clickable((By.XPATH, user_name)))
    driver.find_element_by_xpath(user_name).click()

    dashboard_xpath = "//a[text()='Dashboard']"
    WebDriverWait(driver, 100).until(EC.presence_of_element_located((By.XPATH, dashboard_xpath)))
    time.sleep(20)
    refresh()
    time.sleep(20)
    return driver


def generate_certificate(driver, student_name="Sam", remarks="Completed successfully"):
    # Navigate to certificates
    certificates_xpath = "//a[text()='Certificates']"
    WebDriverWait(driver, 30).until(EC.element_to_be_clickable((By.XPATH, certificates_xpath)))
    driver.find_element_by_xpath(certificates_xpath).click()

    # Select the certificate type
    certificate_type_xpath = "//select[@id='certificate-type']"
    certificate_name_xpath = "//h6[text()='School leaving certificate']"
    WebDriverWait(driver, 30).until(EC.presence_of_element_located((By.XPATH, certificate_type_xpath)))
    Select(driver.find_element_by_xpath(certificate_type_xpath)).select_by_visible_text("School Leaving Certificate")

    # Search and select the student
    search_student_xpath = "//input[@id='student-search']"
    WebDriverWait(driver, 30).until(EC.presence_of_element_located((By.XPATH, search_student_xpath)))
    driver.find_element_by_xpath(search_student_xpath).send_keys(student_name)
    time.sleep(2)  # Allow time for search results to populate

    student_xpath = f"//div[text()='{student_name}']"
    WebDriverWait(driver, 30).until(EC.element_to_be_clickable((By.XPATH, student_xpath)))
    driver.find_element_by_xpath(student_xpath).click()

    # Click on generate
    generate_button_xpath = "//button[@id='generate-certificate-btn']"
    WebDriverWait(driver, 30).until(EC.element_to_be_clickable((By.XPATH, generate_button_xpath)))
    driver.find_element_by_xpath(generate_button_xpath).click()

    # Update remarks
    remarks_xpath = "//textarea[@id='remarks']"
    WebDriverWait(driver, 30).until(EC.presence_of_element_located((By.XPATH, remarks_xpath)))
    driver.find_element_by_xpath(remarks_xpath).send_keys(remarks)

    # Generate and download
    final_generate_button_xpath = "//button[@id='final-generate-btn']"
    WebDriverWait(driver, 30).until(EC.element_to_be_clickable((By.XPATH, final_generate_button_xpath)))
    driver.find_element_by_xpath(final_generate_button_xpath).click()

    # Validate the history of certificates
    history_xpath = "//a[text()='Certificate History']"
    WebDriverWait(driver, 30).until(EC.element_to_be_clickable((By.XPATH, history_xpath)))
    driver.find_element_by_xpath(history_xpath).click()

    # Ensure the certificate is listed in the history
    generated_certificate_xpath = f"//div[text()='{student_name}']/..//div[text()='School Leaving Certificate']"
    WebDriverWait(driver, 30).until(EC.presence_of_element_located((By.XPATH, generated_certificate_xpath)))

    print(f"Successfully generated and validated the School Leaving Certificate for {student_name}")


def main():
    driver = login()
    generate_certificate(driver)
    driver.quit()


if __name__ == "__main__":
    print("start")
    main()
    print("end")

.

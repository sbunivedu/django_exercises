Here's a simple exercise for your students to practice using Django forms in their web apps. This exercise will guide them through creating a form for collecting user data, validating it, and displaying the submitted data.

### Exercise: Build a Contact Form with Django

**Objective:** Create a simple contact form that allows users to submit their name, email address, and a message. When the form is submitted, it should display the data on a new page.

#### Step-by-Step Instructions:

1. **Set up the Django Project:**
   If not already done, create a new Django project and app. You can name the app `contact` or any name of your choice.

   ```bash
   django-admin startproject myproject
   cd myproject
   python manage.py startapp contact
   ```

2. **Create the Form:**
   In the `contact` app, create a form to capture the user's name, email, and message. Create a file called `forms.py` inside the `contact` folder.

   ```python
   # contact/forms.py
   from django import forms

   class ContactForm(forms.Form):
       name = forms.CharField(label="Your Name", max_length=100)
       email = forms.EmailField(label="Your Email")
       message = forms.CharField(label="Your Message", widget=forms.Textarea)
   ```

3. **Create the View:**
   In the `views.py` file of the `contact` app, create a view that will handle both displaying the form and processing the submitted data.

   ```python
   # contact/views.py
   from django.shortcuts import render
   from .forms import ContactForm

   def contact_view(request):
       if request.method == "POST":
           form = ContactForm(request.POST)
           if form.is_valid():
               name = form.cleaned_data['name']
               email = form.cleaned_data['email']
               message = form.cleaned_data['message']
               return render(request, 'contact/thank_you.html', {'name': name, 'email': email, 'message': message})
       else:
           form = ContactForm()
       return render(request, 'contact/contact_form.html', {'form': form})
   ```

4. **Create the Templates:**
   - **Contact Form Template:**

     Create a template called `contact_form.html` where the form will be displayed.

     ```html
     <!-- contact/templates/contact/contact_form.html -->
     <h1>Contact Us</h1>
     <form method="post">
         {% csrf_token %}
         {{ form.as_p }}
         <button type="submit">Submit</button>
     </form>
     ```

   - **Thank You Template:**

     Create a template called `thank_you.html` that will display the user's submitted data.

     ```html
     <!-- contact/templates/contact/thank_you.html -->
     <h1>Thank You, {{ name }}!</h1>
     <p>We have received your message:</p>
     <p><strong>Email:</strong> {{ email }}</p>
     <p><strong>Message:</strong> {{ message }}</p>
     ```

5. **Set up the URL routing:**
   In the `urls.py` file of the `contact` app, create a URL route for the contact view.

   ```python
   # contact/urls.py
   from django.urls import path
   from .views import contact_view

   urlpatterns = [
       path('', contact_view, name='contact'),
   ]
   ```

   In the main `urls.py` of the project, include the `contact` app URLs.

   ```python
   # myproject/urls.py
   from django.contrib import admin
   from django.urls import path, include

   urlpatterns = [
       path('admin/', admin.site.urls),
       path('contact/', include('contact.urls')),
   ]
   ```

6. **Run the server:**
   Start the Django development server.

   ```bash
   python manage.py runserver
   ```

7. **Test the Form:**
   - Visit `http://127.0.0.1:8000/contact/` in your browser.
   - Fill out the contact form and submit it.
   - If the form is valid, it should display the "Thank You" page with the submitted data.

#### Discussion Points:
- Explain how Django handles form validation with `is_valid()` and how `cleaned_data` is used to retrieve the sanitized input data.
- Discuss the use of `CSRF` tokens in forms for security purposes.
- Explore different types of form widgets and how they can be customized for different input types.

This exercise is a good hands-on introduction to working with Django forms, validation, and template rendering.

Here are some suggested answers to the discussion questions:

### 1. **How does Django handle form validation with `is_valid()` and how is `cleaned_data` used to retrieve the sanitized input data?**

   **Answer:**
   - **Form Validation with `is_valid()`:**
     - When a form is submitted, Django first checks whether the form is valid by calling the `is_valid()` method. This method performs the validation on each form field based on the field types and any additional validators specified in the form (e.g., required fields, email format, etc.). If all fields pass their validation checks, `is_valid()` returns `True`; otherwise, it returns `False`.
     - If the form is invalid, the form will contain error messages for the fields that failed validation, which can be accessed and displayed in the template using `form.errors`.

   - **Using `cleaned_data`:**
     - If the form is valid, the `cleaned_data` dictionary holds the sanitized and validated data from the form fields. This dictionary maps the field names (e.g., `'name'`, `'email'`, `'message'`) to their corresponding cleaned values (e.g., stripped of leading/trailing whitespace, converted to the appropriate data types like strings, emails, etc.).
     - For example:
       ```python
       name = form.cleaned_data['name']
       email = form.cleaned_data['email']
       message = form.cleaned_data['message']
       ```
     - `cleaned_data` ensures that the data you work with is secure and properly formatted, preventing issues like XSS or incorrect data types.

### 2. **What is the role of the CSRF token in Django forms, and why is it important?**

   **Answer:**
   - The **CSRF (Cross-Site Request Forgery) token** is a security feature in Django that helps protect web applications from malicious attacks where a third-party website tricks a user into submitting forms on another site without their consent.
   - In a form, you include the CSRF token with `{% csrf_token %}` inside the `<form>` tag. This token is automatically generated by Django when the page is rendered. When the form is submitted, Django compares the token sent with the request to the one stored in the user's session. If they match, the request is considered legitimate. If not, Django will reject the submission and raise a `403 Forbidden` error.
   - **Why it's important:**
     - It prevents attackers from submitting forms on behalf of users without their consent.
     - It ensures that the form submission is coming from the correct site and not a malicious source.

### 3. **What are some different types of form widgets in Django, and how can they be customized?**

   **Answer:**
   - **Form Widgets** are used to render HTML elements for each field in a Django form. The default widgets for most fields are appropriate for general use, but Django also provides ways to customize these widgets for more specific purposes.

   - **Common Widgets:**
     - `forms.CharField` uses a text input widget.
     - `forms.EmailField` uses an email input widget (`<input type="email">`).
     - `forms.DateField` uses a date input widget (`<input type="date">`).
     - `forms.BooleanField` uses a checkbox widget.
     - `forms.ChoiceField` uses a dropdown select widget.

   - **Customizing Widgets:**
     - Widgets can be customized to modify their HTML output, add CSS classes, set placeholder text, etc.
     - Example of customizing a widget:
       ```python
       class ContactForm(forms.Form):
           name = forms.CharField(label="Your Name", max_length=100, widget=forms.TextInput(attrs={'placeholder': 'Enter your name', 'class': 'form-control'}))
           email = forms.EmailField(label="Your Email", widget=forms.EmailInput(attrs={'class': 'form-control'}))
           message = forms.CharField(label="Your Message", widget=forms.Textarea(attrs={'placeholder': 'Your message...', 'rows': 4, 'class': 'form-control'}))
       ```
     - In this example, the `TextInput` and `EmailInput` widgets are customized to include placeholder text and a CSS class (`'form-control'`), and the `Textarea` widget is given a `rows` attribute to control the height of the textarea.

### 4. **How can form validation in Django be extended or customized for complex fields?**

   **Answer:**
   - **Custom Validators:**
     Django allows you to define custom validators to extend the built-in field validation. You can use a custom validator function or a method to validate complex data that doesn't fit the default validation options.

     Example of a custom validator:
     ```python
     from django.core.exceptions import ValidationError

     def validate_email_domain(value):
         if '@example.com' not in value:
             raise ValidationError('Email must belong to the "example.com" domain.')

     class ContactForm(forms.Form):
         email = forms.EmailField(validators=[validate_email_domain])
     ```
     In this example, a custom validator is used to check if the email address contains a specific domain (`example.com`).

   - **Custom Field Clean Methods:**
     You can also override the `clean_<field_name>` method on the form to define custom validation logic for specific fields.

     Example:
     ```python
     class ContactForm(forms.Form):
         message = forms.CharField(label="Your Message", widget=forms.Textarea)

         def clean_message(self):
             message = self.cleaned_data['message']
             if 'spam' in message.lower():
                 raise ValidationError('Message contains spam content.')
             return message
     ```
     In this case, the form checks the `message` field for the word "spam" and raises a validation error if found.

### 5. **What are some best practices for handling forms securely and efficiently in Django?**

   **Answer:**
   - **Use CSRF tokens:** Always include the `{% csrf_token %}` tag in your form to protect against CSRF attacks.
   - **Validate Form Data:** Always use `form.is_valid()` to check that data meets the expected format and constraints.
   - **Sanitize User Input:** Even though Django automatically sanitizes data with `cleaned_data`, always validate and sanitize user input, especially for fields like email, URLs, or custom fields.
   - **Error Handling:** Always display form errors to users in a helpful way, using `{{ form.errors }}` to show validation issues. Custom error messages should be user-friendly.
   - **Limit Form Field Input:** Set appropriate field lengths and use validators to restrict input. For example, use `max_length` for text fields and custom validators for more complex logic.
   - **Use Django Forms for Security and Maintainability:** Using Django’s built-in forms and form validation system allows you to easily manage user input, preventing security issues like SQL injection, XSS, and more.

These practices and concepts will help students build more secure, robust, and maintainable web applications with Django.

# cloud-native-monitoring-app
<img width="568" alt="image" src="https://github.com/134420173-myseneca/cloud-native-monitoring-app/assets/39117847/76ce0bd1-5d71-44c2-9582-0fa205cfabae">

## **Part 1: Deploying the Flask application locally**

### **Step 1: Clone the code**

Clone the code from the repository:

```
git clone https://github.com/134420173-myseneca/cloud-native-monitoring-app.git
```

### **Step 2: Install dependencies**

The application uses the **`psutil`** and **`Flask`, Plotly, boto3** libraries. Install them using pip:

```
pip3 install -r requirements.txt
```

### **Step 3: Run the application**

To run the application, navigate to the root directory of the project and execute the following command:

```
python3 app.py
```

This will start the Flask server on **`localhost:5000`**. Navigate to [http://localhost:5000/](http://localhost:5000/) on your browser to access the application.

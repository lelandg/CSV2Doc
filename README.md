# CSV2Doc

This repository now includes a Streamlit application that allows you to upload a CSV file and convert it to different document formats (CSV, HTML, DOCX). The application provides options to create tables based on column values. Includes advanced group, sort, and filter features.

## CSV to Document Converter

Hosted app: https://csv2doc.streamlit.app/

### Features

- **CSV Upload**: Upload any CSV file for processing
- **Data Preview**: View a preview of your data before conversion
- **Grouping**: Group data by any column to create separate tables for each unique value
- **Multiple Export Formats**: Download your document as CSV, HTML, or DOCX
- **Sorting**: Sort data by any column in ascending or descending order
- **Filtering**: Filter data to include only specific values from a column
- **Custom Document Title**: Set a custom title for your document

### Running the Application Locally

1. Clone the repository.

2. Install the required dependencies. This includes the Streamlit local server and Pandas:
   ```
   pip install -r requirements.txt
   ```

3. Run the Streamlit application:
   ```
   streamlit run app.py
   ```

4. Open your web browser and navigate to the URL displayed in the terminal (usually http://localhost:8501). You'll see the link when you run streamlit.

## Technologies Used

- Python 3.7+
- Streamlit - Web application framework
- Pandas - Data manipulation and analysis
- python-docx - Creating and manipulating DOCX files

## Contributing

Contributions are welcome. Please feel free to submit pull requests with improvements or additions. 

## Problems or Issues

If you encounter any problems or issues, please submit an issue on GitHub.

## License

This project is available under the MIT License.

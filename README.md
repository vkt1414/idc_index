# IDC Index

The IDC Index is a Python library designed to query basic metadata and download data hosted on the NCI Imaging Data Commons (IDC).

## Installation

Install the IDC Index using pip:
```
pip install idc-index
```
## Description

The IDC Index offers a suite of functionalities, enabling users to retrieve diverse information regarding collections, patients, studies, series, and images. The library uses an index of data generated by the following SQL query:

```
SELECT
  STRING_AGG(DISTINCT PatientID) PatientID,
  STRING_AGG(DISTINCT PatientAge) PatientAge,
  STRING_AGG(DISTINCT PatientSex) PatientSex,
  STRING_AGG(DISTINCT collection_id) collection_id,
  STRING_AGG(DISTINCT source_DOI) AS DOI,
  STRING_AGG(DISTINCT StudyInstanceUID) StudyInstanceUID,
  STRING_AGG(DISTINCT CAST(StudyDate AS STRING)) StudyDate,
  STRING_AGG(DISTINCT StudyDescription) StudyDescription,
  STRING_AGG(DISTINCT Modality) Modality,
  STRING_AGG(DISTINCT Manufacturer) Manufacturer,
  STRING_AGG(DISTINCT ManufacturerModelName) ManufacturerModelName,
  SeriesInstanceUID,
  STRING_AGG(DISTINCT CAST(SeriesDate AS STRING)) SeriesDate,
  STRING_AGG(DISTINCT SeriesDescription) SeriesDescription,
  STRING_AGG(DISTINCT BodyPartExamined) BodyPartExamined,
  STRING_AGG(DISTINCT SeriesNumber) SeriesNumber,
  ANY_VALUE(CONCAT("s3://", SPLIT(aws_url,"/")[SAFE_OFFSET(2)], "/", crdc_series_uuid, "/")) AS series_aws_location,
  COUNT(SOPInstanceUID) AS instanceCount,
  ROUND(SUM(instance_size)/(10001000), 2) AS series_size_MB,
FROM
  bigquery-public-data.idc_v16.dicom_all
GROUP BY
  SeriesInstanceUID

```

## Usage

The library provides the following key functionalities along with their available arguments:

- Initialization: Instantiates the IDC Client Class by reading the CSV index and downloading the s5cmd tool.
- IDC Version:
  - get_idc_version() : Get the release version of IDC data 
- Data Retrieval:
  - get_collections(): Retrieve a list of unique collection IDs.
  - get_series_size(seriesInstanceUID): Obtain the size of a series in MB by providing the SeriesInstanceUID.
  - get_patients(collection_id=None, outputFormat="list" or ("dict" or "df")): Retrieve information about patients within a collection.
  - get_dicom_studies(patientId=None, outputFormat="list" or ("dict" or "df")): Retrieve studies for a patient_id.
  - get_dicom_series(studyInstanceUID=None, outputFormat="list" or ("dict" or "df")): Retrieve series within a study.
  - download_dicom_series(seriesInstanceUID, downloadDir, dry_run=False, quiet=True ): Download images associated with a SeriesInstanceUID to a specified directory.
  - download_from_selection(downloadDir=None, dry_run=True, collection_id=None, patientId=None, studyInstanceUID=None): Download images associated with specific filter(s) to a specified directory.
## Resources

To learn more about the IDC, visit Imaging Data Commons at: https://github.com/ImagingDataCommons

For the s5cmd tool used for efficient image retrieval, visit the s5cmd GitHub Repository: https://github.com/peak/s5cmd

## Example

Here's an example demonstrating how to use the IDC Client:


### Initialize the IDC Client
```
from idc_index import index
```
```
idc_client = index.IDCClient()
```
### Check IDC Version
```
idc_client.get_idc_version()
```

### Query data
```
idc_client.get_collections()
```
```
idc_client.get_patients(collection_id='nsclc_radiomics',outputFormat="list")
```
```
idc_client.get_dicom_studies(patientId='D1-0975', outputFormat="dict")
```
```
idc_client.get_dicom_series(studyInstanceUID='1.3.6.1.4.1.32722.99.99.191411096482148278088383576909215626011', outputFormat="df")
```
### Download data
```
idc_client.download_dicom_series(seriesInstanceUID='1.3.6.1.4.1.32722.99.99.459644025247509819689655120845267405', downloadDir='/content/test')
```

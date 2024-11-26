## Info

```scss
.container{
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100%;
    width: 100%;
}
```
### profile image upload
```ts
"use client";
import React, { useState, useEffect } from "react";
import styles from "./styles/profile-image-upload.module.scss";

const ProfileImageUpload = () => {
  const userID = 1;

  // Initial state for image
  const [image, setImage] = useState<File | null>(null);
  const [previewUrl, setPreviewUrl] = useState<string | null>(null);
  const [error, setError] = useState<string | null>(null);
  const [message, setMessage] = useState<string | null>(null);

  // State to hold the current profile image from the server
  const [currentProfileImage, setCurrentProfileImage] = useState<string | null>(
    null
  );

  useEffect(() => {
    if (userID) {
      // Fetch the current profile image URL from the server
      const fetchProfileImage = async () => {
        try {
          const response = await fetch("http://localhost:8006/api/images", {
            credentials: "include",
          });
          if (response.ok) {
            const blob = await response.blob();
            setCurrentProfileImage(URL.createObjectURL(blob)); // Create URL for the image
          } else {
            setCurrentProfileImage(null); // If no profile image, show placeholder
          }
        } catch (err) {
          setCurrentProfileImage(null); // If error, show placeholder
          console.error("Error fetching profile image:", err);
        }
      };

      fetchProfileImage();
    }
  }, [userID]);

  const handleFileChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    if (event.target.files && event.target.files[0]) {
      const selectedImage = event.target.files[0];
      setImage(selectedImage);

      // Create a preview of the selected image
      const reader = new FileReader();
      reader.onloadend = () => {
        setPreviewUrl(reader.result as string); // Set the preview URL
      };
      reader.readAsDataURL(selectedImage); // Read the image as a data URL
    }
  };

  if (!userID) return null;

  const handleUpload = async () => {
    if (!image) {
      setError("Please select an image file first.");
      return;
    }

    const formData = new FormData();
    formData.append("image", image);

    try {
      // Directly uploading to the Spring Boot back-end
      const response = await fetch("http://localhost:8006/api/images/profile", {
        method: "POST",
        credentials: "include",
        body: formData, // This is the multipart form data with the file
      });

      if (!response.ok) {
        throw new Error("Error uploading image.");
      }

      const data = await response.json();
      setMessage("Profile image uploaded successfully!");
    } catch (err) {
      setError("Error uploading image.");
      console.error(err);
    }
  };

  return (
    <div className={styles.container}>
      <h2>Profile Picture</h2>
      {error && <div className={styles.error}>{error}</div>}
      {message && <div className={styles.success}>{message}</div>}

      {/* Image Preview or Profile Image */}
      <div
        className={`${styles.imageContainer} ${
          currentProfileImage || previewUrl ? "" : styles.placeholder
        }`}
      >
        {previewUrl || currentProfileImage ? (
          <img
            src={previewUrl || currentProfileImage || ""} // Default placeholder if no image
            alt="Profile"
            className={styles.image}
          />
        ) : (
          <i className="bx bx-user"></i>
        )}
      </div>

      {/* File Input */}
      <input type="file" id="image" onChange={handleFileChange} hidden />
      <label htmlFor="image" className={styles.label}>
        Choose new image
      </label>

      {/* Upload Button */}
      <button
        onClick={handleUpload}
        className={styles.button}
        disabled={!previewUrl}
      >
        Save Image
      </button>
    </div>
  );
};

export default ProfileImageUpload;
```

```scss
.form{
    width: max-content;

    padding: 10px;

    .section{
        padding: 10px;
        margin: 10px 0;
        background-color: white;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        width: 100%;
    
        
    
    }
}
```


### dynamic form
```ts
"use client";
import { useFetch } from "@/app/context/FetchProvider";
import InputField from "@/app/shared/input-field";
import SelectField from "@/app/shared/select-field";
import { generateFormObject, prepareRequest } from "@/app/shared/utils";

import { Form, Formik } from "formik";
import React, { useEffect } from "react";
import styles from "./styles/dynamic-form.module.scss";

interface DynamicFormProps {
  sections: Section[];
}

const DynamicForm = (props: DynamicFormProps) => {
  const userID = 1;
  const { result, metadata } = generateFormObject(props.sections);
  const { post } = useFetch();

  if (Object.keys(result).length === 0) return null;

  const submitValues = async (values: any) => {
    const retrive = await post("api/values", {
      userID,
      values: prepareRequest(values, metadata),
    });
    if (retrive) {
      // successful
    }
  };

  return (
    <Formik
      initialValues={result}
      onSubmit={async (values, { setSubmitting }) => {
        await submitValues(values);
        setSubmitting(false);
      }}
    >
      {({ isSubmitting }) => (
        <Form className={styles.form} >
          {props.sections &&
            props.sections.map((s, idx) => {
              return (
                <div
                className={styles.section}
                  key={`s#${idx}`}
                >
                  <h1> {s.name} </h1>
                  {s.fields.map((f, i) => {
                    if (f.type === "OPTIONS") {
                      return (
                        <SelectField
                          key={`f#${i}`}
                          label={f.name}
                          name={`${s.name}.${f.name}`}
                          options={f.options}
                        />
                      );
                    } else if (f.type === "COMPOSITE") {
                      return (
                        <div key={`f#${i}`} style={{ marginLeft: "10px" }}>
                          <h3>{f.name}</h3>
                          {f.compositeFields.map((cf, ci) => {
                            return cf.type === "OPTIONS" ? (
                              <SelectField
                                key={`cf#${ci}`}
                                label={cf.name}
                                name={`${s.name}.${f.name}.${cf.name}`}
                                options={cf.options || []}
                              />
                            ) : (
                              <InputField
                                key={`cf#${ci}`}
                                label={cf.name}
                                name={`${s.name}.${f.name}.${cf.name}`}
                                type={cf.type === "DATE" ? "date" : "text"}
                              />
                            );
                          })}
                        </div>
                      );
                    } else {
                      return (
                        <InputField
                          key={`f#${i}`}
                          label={f.name}
                          name={`${s.name}.${f.name}`}
                          type={f.type === "DATE" ? "date" : "text"}
                        />
                      );
                    }
                  })}
                </div>
              );
            })}

          <button type="submit" disabled={isSubmitting}>
            Save Info
          </button>
        </Form>
      )}
    </Formik>
  );
};

export default DynamicForm;
```

```scss
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.error {
  color: red;
}

.success {
  color: green;
}

.imageContainer {
  width: 150px;
  height: 150px;
  overflow: hidden;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 50%; /* Optional: make it circular */
}

.placeholder {
  background-color: #d3d3d3; /* Grey background */
}

.image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.label {
  display: inline-block;
  background-color: #007bff;
  color: white;
  padding: 10px 20px;
  font-size: 16px;
  border-radius: 5px;
  cursor: pointer;
  text-align: center;

  &:hover {
    background-color: #0056b3;
  }
}

.button {
  background-color: #28a745;
  color: white;
  padding: 10px 20px;
  font-size: 16px;
  border: none;
  border-radius: 5px;
  cursor: pointer;

  &:hover {
    background-color: #218838;
  }

  &:disabled {
    background-color: #6c757d;
    cursor: not-allowed;
  }
}

```



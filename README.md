# IEEE-CIS Fraud Detection

## Kaggle-ის კონკურსის მოკლე მიმოხილვა  
IEEE-CIS Fraud Detection კონკურსში ჩვენი მიზანია გავარჩიოთ ონლაინ ტრანზაქციების მართალი და ყალბი (fraud) შემთხვევები. ვარჩევთ Vesta Corporation–ის მონაცემებს, სადაც თითოეულ ტრანზაქციას აქვს ორი ნაწილი—`transaction` და `identity` და ბინარული ლეიბლი `isFraud` (0/1). შეფასება ხდება ROC AUC–ით.

## ჩემი მიდგომა პრობლემის გადასაჭრელად  
1. **Data Merge & Cleaning**  
   - `transaction` და `identity` დავაჯოინე TransactionID-ით  
   - >90% nan ებით შევსებული Feature-ების მოშორება  
   - nan ების ამოშლა–შევსება:  
     - რიცხობრივი ცვლადები: median  
     - კატეგორიული ცვლადები: ნიშანი `"missing"`  
2. **Feature Engineering**  
   - კატეგორიების დასადგენად  
     - უნიკალური მნიშვნელობა ≤3 → One-Hot Encoding  
     - უნიკალური >3 → WoE Encoding   
3. **Feature Selection**  
   - SHAP არეგულარული მოდელით 
   - კრიტერიუმი: იმპორტანსის საერთო მნიშვნელობის 95% შემადგენელ ცვლადებს ვიტოვებდი
4. **Model Training & Tuning**  
   - ქროს-ვალიდაცია, GridSearchCV თითოეულ ალგორითმზე.
   - **Logistic Regression** (C ∈ {0.01, 0.1, 1, 10} -> 1)  
   - **Random Forest**  
     - max_depth ∈ {10, 15, 20} → 20  
     - n_estimators ∈ {100, 200, 300, 500} → 500  
     - min_samples_split ∈ {2, 5, 10, 20} → 2  
     - min_samples_leaf ∈ {1, 2, 4, 8} → 2  
   - **XGBoost**  
     - max_depth ∈ {5, 10, 15} → 15  
     - learning_rate ∈ {0.05, 0.1, 0.2, 0.35, 0.5} → 0.2  
     - n_estimators ∈ {50, 100, 200, 350, 500, 750, 1000} → 500  
5. **Model Selection**  
   XGBoost აჩვენა საუკეთესო ბალანსი გაზრდილი train AUC და დაბალი overfit–ი.

## რეპოზიტორიის სტრუქტურა  

```
├── notebooks/                            # ექსპერიმენტული Jupyter Notebook-ები და მოდელის Inference
│   ├── model_experiment_Logistic.ipynb      # Logistic Regression ექსპერიმენტი
│   ├── model_experiment_RandomForest.ipynb  # Random Forest ექსპერიმენტი
│   ├── model_experiment_XGBoost.ipynb       # XGBoost ექსპერიმენტი
│   └── model_inference.ipynb                # Model Registry-დან საუკეთესო მოდელის ჩატვირთვა და submission.csv გენერაცია
└── README.md                               # ამ ფაილის აღწერა
```


### ყველა ფაილის განმარტება   
- **notebooks/**:  
  - `model_experiment_*.ipynb` — თითოეული არქიტექტურისთვის ცალკე ექსპერიმენტი  
  - `model_inference.ipynb` — Model Registry–დან მოდელის ჩატვირთვა, test set პროგნოზი, Kaggle submission  


---

## Feature Engineering

### კატეგორიული ცვლადების რიცხვითში გადაყვანა  
- უნიკალური მნიშნველობა ≤3 → `OneHotEncoder()`  
- უნიკალური მნიშვნელობა >3 → WoE (Weight of Evidence) encoding  

### Nan მნიშვნელობების დამუშავება  
- რაოდენობრივი ცვლადები: median–ით  
- კატეგორიული ცვლადები: ახალი კატეგორია `"missing"`  

### Cleaning მიდგომები  
- >90% nan მნიშვნელობა → ვაშორებ ცვლადს , რადგან არანაირი რეალური შედეგი არ ექნებოდა ამ ცვლადს მოდელზე, ძირითად შემთხვევებში ისედაც ნანია და უბრალოდ დაამძიმებდა მოდელის ტრენინგს  

---

## Feature Selection

### გამოყენებული მიდგომები და მათი შეფასება  
1. **SHAP** (TreeExplainer) — ქმედითი ფიელდების აწყობა , ცვლადები რომლებიც გვაძლევენ საერთო იმპორტანსის 95% საკმარისია რომ დავიტოვოთ და მოვაშოროთ დანარჩენი, რათა უფრო ეფექტური გავხადოთ მოდელის ტრენინგი, მით უმეტეს როცა ამხელა დატასთან გვაქვს საქმე 
3. **IV (Information Value)** — გამოიყენება WOE კლასში დამატებით  

---

## Training

### ტესტირებული მოდელები

#### 1. Logistic Regression  
- **პარამეტრები**: C = 1 (best)  
- **მეტრიკა (Validation)**:  
  - F1: 0.467  
  - Accuracy: 0.963  
  - Recall: 0.467  
  - Precision: 0.467  
  - ROC AUC: 0.872  
- **ანალიზი**:  
  - შედარებით დაბალი expressiveness → underfitting, რთული პრობლემა იყო და წრფივი დაყოფა მრავლის მომცველი ვერ აღმოჩნდა ურთიერთობებისთვი შესაბამისად მალევე შევეშვი და გადავედი უფრო კომპექსურ შემდეგ მოდელებზე.  

#### 2. Random Forest  
- **პარამეტრები**:  
  - max_depth = 20, n_estimators = 500  
  - min_samples_split = 2, min_samples_leaf = 2  
- **Train Metrics**:  
  - Accuracy: 0.981  
  - Precision: 0.765  
  - Recall: 0.661  
  - F1: 0.709  
  - ROC AUC: 0.951  
- **Test Metrics**:  
  - Accuracy: 0.976  
  - Precision: 0.689  
  - Recall: 0.551  
  - F1: 0.612  
  - ROC AUC: 0.918  
- **ანალიზი**:  
  - შედარებით მაღალი train vs test განსხვავება → მინიმალური underfitting, შედარებით ჯენერალიზებული, ყველა ჰიპერპარამეტრი დატუნინგებულია გრიდსერჩის მიერ, გაცილებით უკეთესი შედეგი მოგვცა ვიდრე ჩვეულებრივმა ლოჯისტიკ რეგრეშენმა რასაც ისედაც მოველოდით. 

#### 3. XGBoost (Final)  
- **პარამეტრები**:  
  - max_depth = 15, learning_rate = 0.2, n_estimators = 500  
- **Train Metrics**:  
  - Accuracy: 0.991  
  - Precision: 0.803  
  - Recall: 1.000  
  - F1: 0.891  
  - ROC AUC: 0.999  
- **Test Metrics**:  
  - Accuracy: 0.982  
  - Precision: 0.725  
  - Recall: 0.795  
  - F1: 0.759  
  - ROC AUC: 0.971  
- **ანალიზი**:  
  - მაღალი train AUC მიუთითებს  რაღაც overfit ზე, მაგრამ test შედეგები საუკეთესოდ ჯენერალიზებულია და რეალურ პრობლემასთან სამუშაოდ კარგი მოდელია, ეს მოდელი აღმოჩნდა ასევე საუკეთესო ჩემს მიერ გარჩეულ მოდელებს შორის რაც ასევე ნავარაუდევი იყო.

### Hyperparameter ოპტიმიზაციის მიდგომა  
- შერჩეული პარამეტრების გადატესტვა `GridSearchCV`–ით K-Fold CV (k=2)  
- ყოველი მოდელისთვის გამოვყავი X_sample და y_sample რადგან თვითონ ჰიპერპარამეტრების ტუნინგი გამეკეთებინა 20-50% დატაზე, რადგან X_train 80% ძალიან დიდ დატასთან გვექნებოდა საქმე და ამან ხშირად გადამიტვირთა კაგლის რამი. ტუნინგის მერე ჩვეულებრივად 80-20 split ით ვატრეინებდი და ვტესტავდი მოდელს
- თითოეულ ექსპერიმენტს MLflow–ზე დალოგილი აქვს გრაფები(ROC curve), მეტრიკები და ჰიპერპარამეტრები 

### საბოლოო მოდელის შერჩევის დასაბუთება  
XGBoostმა გამოიჩინა საუკეთესო კომბინაცია მაღალი AUC და საკმარისი F1, მიუხედავად იმისა რომ შედარებით overfitted მოდელი არის, რეალურ პრობლემაზე სამუშაოდ საკმაო სიძლიერე გამოაჩინა ტესტ დატაზე, შესაბამისად გადავწყვიტე კაგლზე საბმიშენად სწორედ ეს მოდელი გამეშვა და კარგი შედეგიც დავდე

---

## MLflow Tracking

- **Dagshub MLflow URI**:  
  https://dagshub.com/losaberidzebadri/IEEE-CIS-Fraud-Detection.mlflow 
- **Experiments**:  
  - `LogisticRegressionTraining`  
  - `RandomForestTraining`  
  - `XGBoostTraining`  
- **ჩაწერილი მეტრიკები**:  
  - ROC AUC, Accuracy, Precision, Recall, F1  
- **საუკეთესო მოდელის შედეგები**:  
  - XGBClassifier_BestParams → Test AUC: 0.971, Test F1: 0.759  

---

**Kaggle Submission**  
- Public LB: **0.930**  
- Private LB: **0.890**  

---


# Running Containers and Tests  

## Starting and Managing Containers  

### Start Containers  
```sh
cd ~/sssd-ci-containers/  
sudo make up  
```  

## Running Active Directory (AD)  

### Start AD  
```sh
cd ~/sssd-ci-containers/src  
vagrant up  
```  

### Stop AD  
```sh
vagrant down  
```  

### Remove AD  
```sh
vagrant destroy  
```  

## Running System Tests  

### Run All Tests  
```sh
cd ~/sssd/src/tests/system  
sudo pytest --mh-config=mhc.yaml --mh-lazy-ssh -v  
```  

### Run Specific Tests (e.g., IPA, AD, Samba, Client)  
```sh
sudo pytest --mh-config=mhc.yaml --mh-topology=ldap  
```  

## Test Results Interpretation  

- `.` → 1 test passed  
- `F` → 1 test failed  


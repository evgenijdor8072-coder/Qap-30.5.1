import pytest
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

class TestUserPetsPage:
    
    def test_all_pets_are_present(self, driver):
        """Тест 1: Присутствуют все питомцы."""
        pets = WebDriverWait(driver, 10).until(
            EC.presence_of_all_elements_located((By.CSS_SELECTOR, ".pet-card"))
        )
        assert len(pets) > 0, "На странице нет питомцев"

    def test_at_least_half_of_pets_have_photos(self, driver):
        """Тест 2: Хотя бы у половины питомцев есть фото."""
        pets = driver.find_elements(By.CSS_SELECTOR, ".pet-card")
        photo_count = 0
        
        for pet in pets:
            img = pet.find_element(By.TAG_NAME, "img")
            src = img.get_attribute("src")
            if src and src != "" and "placeholder" not in src:
                # Дополнительная проверка, что изображение действительно загружено
                # (можно проверить размеры через JavaScript)
                photo_count += 1
        
        assert photo_count >= len(pets) / 2, \
            f"Фото есть только у {photo_count} из {len(pets)} питомцев. Требуется хотя бы у {len(pets) // 2 + 1}"

    def test_all_pets_have_name_age_breed(self, driver):
        """Тест 3: У всех питомцев есть имя, возраст и порода."""
        pets = driver.find_elements(By.CSS_SELECTOR, ".pet-card")
        
        for i, pet in enumerate(pets):
            name_element = pet.find_element(By.CSS_SELECTOR, "h5")
            age_element = pet.find_element(By.XPATH, ".//p[contains(text(), 'Возраст')]")
            breed_element = pet.find_element(By.XPATH, ".//p[contains(text(), 'Порода')]")
            
            name = name_element.text.strip()
            age = age_element.text.replace("Возраст:", "").strip()
            breed = breed_element.text.replace("Порода:", "").strip()
            
            assert name != "", f"У питомца {i+1} отсутствует имя"
            assert age != "", f"У питомца {i+1} отсутствует возраст"
            assert breed != "", f"У питомца {i+1} отсутствует порода"

    def test_all_pets_have_different_names(self, driver):
        """Тест 4: У всех питомцев разные имена."""
        pets = driver.find_elements(By.CSS_SELECTOR, ".pet-card")
        names = []
        
        for pet in pets:
            name_element = pet.find_element(By.CSS_SELECTOR, "h5")
            name = name_element.text.strip()
            names.append(name)
        
        unique_names = set(names)
        assert len(names) == len(unique_names), \
            f"Найдены повторяющиеся имена: {[name for name in names if names.count(name) > 1]}"

    def test_no_duplicate_pets(self, driver):
        """Тест 5: В списке нет повторяющихся питомцев."""
        pets = driver.find_elements(By.CSS_SELECTOR, ".pet-card")
        pets_data = []
        
        for pet in pets:
            name_element = pet.find_element(By.CSS_SELECTOR, "h5")
            age_element = pet.find_element(By.XPATH, ".//p[contains(text(), 'Возраст')]")
            breed_element = pet.find_element(By.XPATH, ".//p[contains(text(), 'Порода')]")
            
            name = name_element.text.strip()
            age = age_element.text.replace("Возраст:", "").strip()
            breed = breed_element.text.replace("Порода:", "").strip()
            
            pets_data.append((name, age, breed))
        
        seen = set()
        duplicates = []
        
        for pet_data in pets_data:
            if pet_data in seen:
                duplicates.append(pet_data)
            seen.add(pet_data)
        
        assert len(duplicates) == 0, \
            f"Найдены повторяющиеся питомцы: {duplicates}"

def test_user_pets_statistics(driver):
    """Проверка статистики питомцев пользователя."""
    driver.get("https://your-site.com/user/profile")
    
    test_class = TestUserPetsPage()
    
    test_class.test_all_pets_are_present(driver)
    test_class.test_at_least_half_of_pets_have_photos(driver)
    test_class.test_all_pets_have_name_age_breed(driver)
    test_class.test_all_pets_have_different_names(driver)
    test_class.test_no_duplicate_pets(driver)
